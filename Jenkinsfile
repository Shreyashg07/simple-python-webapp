pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Shreyashg07/simple-python-webapp.git'
            }
        }

        stage('Install') {
            steps {
                sh '''
                    python3 -m venv venv
                    . venv/bin/activate
                    pip install --upgrade pip
                    pip install -r requirements.txt
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                    . venv/bin/activate
                    pytest --maxfail=1 --disable-warnings -q
                '''
            }
        }

        stage('Trivy Scan') {
            steps {
                echo '🔍 Running Trivy vulnerability scan...'
                sh '''
                    mkdir -p trivy-reports
                    # Run scan using config file if exists, else default to fs scan
                    if [ -f trivy.yaml ]; then
                        trivy fs --config trivy.yaml
                    else
                        trivy fs --format json -o trivy-reports/trivy-report.json .
                    fi

                    # Generate human-readable HTML report
                    trivy fs --format template --template "@contrib/html.tpl" -o trivy-reports/trivy-report.html .
                '''
            }
            post {
                always {
                    echo '📦 Archiving Trivy reports...'
                    archiveArtifacts artifacts: 'trivy-reports/*.json', fingerprint: true
                    archiveArtifacts artifacts: 'trivy-reports/*.html', fingerprint: true
                }
            }
        }
    }

    post {
        always {
            echo '✅ Pipeline completed (with or without issues).'
        }
    }
}
