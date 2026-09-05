pipeline {
    agent any

    environment {
        IMAGE_NAME = "devops-assignment-app"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out source code'
            }
        }

        stage('Validate') {
            steps {
                sh '''
                    echo "Validating project..."

                    test -f app/Dockerfile
                    test -f app/app.py
                    test -f app/requirements.txt
                    test -f helm/devops-app/Chart.yaml
                    test -f app/tests/test_app.py

                    echo "Validation successful"
                '''
            }
        }

        stage('Unit Test') {
            steps {
                sh '''
                    echo "Running unit tests..."

                    python3 -m venv venv-ci
                    . venv-ci/bin/activate

                    pip install --upgrade pip
                    pip install -r app/requirements.txt

                    PYTHONPATH=. python -m pytest app/tests

                    echo "Unit tests passed"
                '''
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    echo "Building Docker image..."

                    docker build \
                        --platform linux/amd64 \
                        -t ${IMAGE_NAME}:${IMAGE_TAG} \
                        ./app

                    echo "Docker image built successfully"

                    docker images ${IMAGE_NAME}:${IMAGE_TAG}
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