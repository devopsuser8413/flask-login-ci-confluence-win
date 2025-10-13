pipeline {
    agent any

    environment {
        // Credentials
        SMTP_HOST       = credentials('smtp-host')
        SMTP_PORT       = '587'
        SMTP_USER       = credentials('smtp-user')
        SMTP_PASS       = credentials('smtp-pass')
        CONFLUENCE_BASE  = credentials('confluence-base')
        CONFLUENCE_USER  = credentials('confluence-user')
        CONFLUENCE_TOKEN = credentials('confluence-token')
        CONFLUENCE_SPACE = 'DEMO'
        CONFLUENCE_TITLE = 'CI Test Report'
        GITHUB_CREDENTIALS = credentials('github-credentials')

        // Paths
        REPORT_PATH = 'report\\report.html'
        VENV_PATH   = '.venv'
    }

    stages {
        stage('Setup Python Environment') {
            steps {
                bat '''
                    @echo off
                    echo Checking Python...
                    python --version

                    echo Creating virtual environment if it doesn't exist...
                    if not exist "%VENV_PATH%" (
                        python -m venv %VENV_PATH%
                    )

                    echo Python & pip in venv:
                    %VENV_PATH%\\Scripts\\python.exe --version
                    %VENV_PATH%\\Scripts\\pip.exe --version
                '''
            }
        }

        stage('Checkout from GitHub') {
            steps {
                echo 'Checking out source code from GitHub repository...'
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[
                        url: 'https://github.com/devopsuser8413/flask-login-ci-confluence-win.git',
                        credentialsId: 'github-credentials'
                    ]]
                ])
            }
        }

        stage('Install Dependencies') {
            steps {
                bat """
                    echo Installing dependencies...
                    %VENV_PATH%\\Scripts\\python.exe -m pip install --upgrade pip
                    %VENV_PATH%\\Scripts\\python.exe -m pip install -r requirements.txt
                """
            }
        }

        stage('Run Tests') {
            steps {
                bat """
                    echo Running tests...
                    if not exist "report" mkdir report
                    set PYTHONPATH=%CD%
                    %VENV_PATH%\\Scripts\\python.exe -m pytest --html=%REPORT_PATH% --self-contained-html || exit /b 0
                """
            }
            post {
                always {
                    archiveArtifacts artifacts: 'report\\report.html', fingerprint: true
                }
            }
        }

        stage('Test Confluence API') {
            steps {
                bat """
                    echo Verifying Confluence API...
                    set PYTHONUTF8=1
                    %VENV_PATH%\\Scripts\\python.exe check_api_token.py
                """
            }
        }

        stage('Email Report') {
            steps {
                bat """
                    echo Sending test report via email...
                    set PYTHONUTF8=1
                    %VENV_PATH%\\Scripts\\python.exe send_report_email.py
                """
            }
        }

        stage('Publish to Confluence') {
            steps {
                bat """
                    echo Publishing HTML report to Confluence...
                    set PYTHONUTF8=1
                    %VENV_PATH%\\Scripts\\python.exe publish_to_confluence.py
                """
            }
        }
    }

    post {
        success { echo 'Pipeline completed successfully!' }
        failure { echo 'Pipeline failed. Check logs!' }
    }
}
