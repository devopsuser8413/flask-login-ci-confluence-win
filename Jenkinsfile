pipeline {
    agent any

    environment {
        // Python Path
        PYTHON = 'C:\\Users\\I17270834\\AppData\\Local\\Programs\\Python\\Python313\\python.exe'

        // Credentials
        SMTP_HOST        = credentials('smtp-host')
        SMTP_PORT        = '587'
        SMTP_USER        = credentials('smtp-user')
        SMTP_PASS        = credentials('smtp-pass')
        CONFLUENCE_BASE  = credentials('confluence-base')
        CONFLUENCE_USER  = credentials('confluence-user')
        CONFLUENCE_TOKEN = credentials('confluence-token')
        CONFLUENCE_SPACE = 'DEMO'
        CONFLUENCE_TITLE = 'CI Test Report'
        GITHUB_CREDENTIALS = credentials('github-credentials')

        // Paths
        VENV_PATH   = '.venv'
        REPORT_PATH = 'report\\report.html'
    }

    stages {
        stage('Install Python if Missing') {
            steps {
                bat """
                    echo Checking if Python is installed...
                    where python || echo Python not found!
                    "${PYTHON}" --version
                    "${PYTHON}" -m pip --version
                """
            }
        }

        stage('Setup Python Environment') {
            steps {
                bat """
                    echo Creating virtual environment if it doesn't exist...
                    if not exist "%VENV_PATH%" (
                        "${PYTHON}" -m venv %VENV_PATH%
                    )
                    echo Activating virtual environment...
                    call %VENV_PATH%\\Scripts\\activate
                    echo Verifying Python in venv:
                    python --version
                """
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
                    call %VENV_PATH%\\Scripts\\activate
                    echo Installing Python dependencies...
                    python -m pip install --upgrade pip
                    python -m pip install -r requirements.txt
                """
            }
        }

        stage('Run Tests') {
            steps {
                bat """
                    call %VENV_PATH%\\Scripts\\activate
                    echo Running tests and generating HTML report...
                    python -m pytest --html=%REPORT_PATH% --self-contained-html || exit /b 0
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
                    call %VENV_PATH%\\Scripts\\activate
                    echo Verifying Confluence API connection...
                    set PYTHONUTF8=1
                    python check_api_token.py
                """
            }
        }

        stage('Publish to Confluence') {
            steps {
                bat """
                    call %VENV_PATH%\\Scripts\\activate
                    echo Publishing test report to Confluence...
                    set PYTHONUTF8=1
                    python publish_to_confluence.py
                """
            }
        }

        stage('Email Report') {
            steps {
                bat """
                    call %VENV_PATH%\\Scripts\\activate
                    echo Sending test report via email...
                    set PYTHONUTF8=1
                    python send_report_email.py
                """
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline completed successfully!'
        }
        failure {
            echo '❌ Pipeline failed. Check Jenkins console logs for details.'
        }
    }
}
