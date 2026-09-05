pipeline {
    agent any

    environment {
        AWS_REGION = "ap-south-1"
        ECR_REPOSITORY = "584523978049.dkr.ecr.ap-south-1.amazonaws.com/devops-assignment-app"

        IMAGE_NAME = "devops-assignment-app"
        IMAGE_TAG = "${BUILD_NUMBER}"

        PATH = "/Users/surajpatil/.docker/bin:/opt/homebrew/bin:${env.PATH}"
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
                    test -f app/tests/test_app.py
                    test -f helm/devops-app/Chart.yaml

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

        stage('Trivy Scan') {
            steps {
                sh '''
                    echo "Checking Trivy installation..."

                    trivy --version

                    echo "Scanning Docker image with Trivy..."

                    trivy image \
                        --severity HIGH,CRITICAL \
                        ${IMAGE_NAME}:${IMAGE_TAG}

                    echo "Trivy scan completed"
                '''
            }
        }

        stage('ECR Login') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'aws-ecr-credentials',
                        usernameVariable: 'AWS_ACCESS_KEY_ID',
                        passwordVariable: 'AWS_SECRET_ACCESS_KEY'
                    )
                ]) {
                    sh '''
                        echo "Logging in to Amazon ECR..."

                        aws ecr get-login-password \
                            --region ${AWS_REGION} | \
                            docker login \
                            --username AWS \
                            --password-stdin ${ECR_REPOSITORY}

                        echo "ECR login successful"
                    '''
                }
            }
        }

        stage('Push Image to ECR') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'aws-ecr-credentials',
                        usernameVariable: 'AWS_ACCESS_KEY_ID',
                        passwordVariable: 'AWS_SECRET_ACCESS_KEY'
                    )
                ]) {
                    sh '''
                        echo "Tagging Docker image for ECR..."

                        docker tag \
                            ${IMAGE_NAME}:${IMAGE_TAG} \
                            ${ECR_REPOSITORY}:${IMAGE_TAG}

                        echo "Pushing image to ECR..."

                        docker push \
                            ${ECR_REPOSITORY}:${IMAGE_TAG}

                        echo "Docker image pushed successfully"
                    '''
                }
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