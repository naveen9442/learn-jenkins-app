pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                echo 'Building application...'
                sh 'ls -a'
            }
        }

        stage('Test') {
            steps {
                echo 'Running tests...'
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying application...'
            }
        }
    }
}