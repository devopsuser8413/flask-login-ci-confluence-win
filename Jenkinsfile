pipeline {
    agent any

    environment {
        // 🔐 SMTP credentials for sending test report emails
        SMTP_HOST       = credentials('smtp-host')
        SMTP_PORT       = '587'
        SMTP_USER       = credentials('smtp-user')
        SMTP_PASS       = credentials('smtp-pass')

        // 🔐 Confluence credentials for publishing report
        CONFLUENCE_BASE  = credentials('confluence-base')
        CONFLUENCE_USER  = credentials('confluence-user')
        CONFLUENCE_TOKEN = credentials('confluence-token')
        CONFLUENCE_SPACE = 'DEMO'
        CONFLUENCE_TITLE = 'CI Test Report'

        // 🧩 GitHub repository credentials for private repo access
        GITHUB_CREDENTIALS = credentials('github-credentials')

        // 📄 Test report file path
        REPORT_PATH = 'report/report.html'
        
        // Ensure Python from venv is used in all stages
        VENV_PATH   = '.venv'
        PATH        = "${WORKSPACE}/.venv/bin:${env.PATH}"
    }

    stages {
        stage('Setup Python Environment') {
            steps {
                sh '''
                    set -eux

                    echo "🐍 Checking Python..."
                    python3 --version

                    echo "📦 Bootstrapping pip and virtualenv..."
                    curl -sS https://bootstrap.pypa.io/get-pip.py -o get-pip.py
                    python3 get-pip.py --user --break-system-packages || true
                    python3 -m pip install --user virtualenv --break-system-packages || true

                    echo "🏗️ Creating local virtual environment..."
                    if [ ! -d "${VENV_PATH}" ]; then
                        ~/.local/bin/virtualenv ${VENV_PATH} || python3 -m venv ${VENV_PATH} || true
                    fi

                    echo "✅ Python & pip inside venv:"
                    python --version
                    pip --version
                '''
            }
        }

        stage('Checkout from GitHub') {
            steps {
                echo '📥 Checking out source code from GitHub repository...'
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[
                        url: 'https://github.com/devopsuser8413/flask-login-ci-confluence.git',
                        credentialsId: 'github-credentials'
                    ]]
                ])
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    echo "📦 Installing dependencies..."
                    pip install --upgrade pip --break-system-packages
                    pip install -r requirements.txt --break-system-packages
                '''
            }
        }

        stage('Run Tests') {
            steps {
                sh '''
                    echo "🧪 Running tests..."
                    mkdir -p report
                    PYTHONPATH=$PWD pytest --html=${REPORT_PATH} --self-contained-html || true
                '''
            }
            post {
                always {
                    archiveArtifacts artifacts: 'report/report.html', fingerprint: true
                }
            }
        }

        stage('Test Confluence API') {
            steps {
                echo '🔎 Verifying Confluence API token and permissions using check_api_token.py...'
                sh 'python check_api_token.py'
            }
        }

        stage('Email Report') {
            steps {
                echo '📧 Sending test report via email...'
                sh 'python send_report_email.py'
            }
        }

        stage('Publish to Confluence') {
            steps {
                echo '🌐 Publishing HTML report to Confluence page...'
                sh 'python publish_to_confluence.py'
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline completed successfully! Report emailed and published to Confluence.'
        }
        failure {
            echo '❌ Pipeline failed. Check the Jenkins console output for details.'
        }
    }
}
