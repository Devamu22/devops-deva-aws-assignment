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
                    test -f app/Dockerfile
                    test -f app/app.py
                    test -f helm/devops-app/Chart.yaml
                    echo "Validation successful"
                '''
            }
        }

        stage('Unit Test') {
    steps {
        sh '''
            python3 -m venv venv-ci
            . venv-ci/bin/activate

            pip install --upgrade pip
            pip install -r app/requirements.txt
            pip install pytest

            pytest app/tests
        '''
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