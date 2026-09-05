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
                    echo "========================================"
                    echo "Validating project structure"
                    echo "========================================"

                    test -f app/Dockerfile
                    test -f app/app.py
                    test -f app/requirements.txt
                    test -f app/tests/test_app.py
                    test -f helm/devops-app/Chart.yaml
                    test -f helm/devops-app/values.yaml

                    echo "Validation successful"
                '''
            }
        }

        stage('Unit Test') {
            steps {
                sh '''
                    echo "========================================"
                    echo "Running unit tests"
                    echo "========================================"

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
                    echo "========================================"
                    echo "Building Docker image"
                    echo "========================================"

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
                    echo "========================================"
                    echo "Running Trivy image scan"
                    echo "========================================"

                    trivy --version

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
                        echo "========================================"
                        echo "Logging in to Amazon ECR"
                        echo "========================================"

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

        stage('ECR Push') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'aws-ecr-credentials',
                        usernameVariable: 'AWS_ACCESS_KEY_ID',
                        passwordVariable: 'AWS_SECRET_ACCESS_KEY'
                    )
                ]) {
                    sh '''
                        echo "========================================"
                        echo "Tagging Docker image"
                        echo "========================================"

                        docker tag \
                            ${IMAGE_NAME}:${IMAGE_TAG} \
                            ${ECR_REPOSITORY}:${IMAGE_TAG}

                        echo "Pushing image to ECR"

                        docker push \
                            ${ECR_REPOSITORY}:${IMAGE_TAG}

                        echo "ECR push successful"
                    '''
                }
            }
        }

        stage('Helm Lint') {
            steps {
                sh '''
                    echo "========================================"
                    echo "Running Helm lint"
                    echo "========================================"

                    helm lint ./helm/devops-app

                    echo "Helm lint passed"
                '''
            }
        }

        stage('Helm Template') {
            steps {
                sh '''
                    echo "========================================"
                    echo "Rendering Helm templates"
                    echo "========================================"

                    helm template devops-app ./helm/devops-app

                    echo "Helm template rendering successful"
                '''
            }
        }
    }

    post {

        success {
            echo '''
            ========================================
            PIPELINE COMPLETED SUCCESSFULLY
            ========================================
            '''
        }

        failure {
            echo '''
            ========================================
            PIPELINE FAILED
            ========================================
            Please check the failed stage above.
            ========================================
            '''
        }

        always {
            echo "Jenkins build completed: ${BUILD_NUMBER}"
        }
    }
}