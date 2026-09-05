pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out source code'
            }
        }

        stage('Validate') {
            steps {
                sh '''
                    test -f Dockerfile
                    test -f app/app.py
                    test -f helm/devops-app/Chart.yaml
                    echo "Validation successful"
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                    python3 -m py_compile app/app.py
                    echo "Tests passed"
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully'
        }
        failure {
            echo 'Pipeline failed'
        }
    }
}