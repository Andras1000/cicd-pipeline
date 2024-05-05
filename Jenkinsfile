pipeline {
    agent any
    environment {
        NODEJS_INSTALLATION = 'node'
    }
    tools {
        nodejs NODEJS_INSTALLATION
    }
    stages {
        stage ('nodejs test') {
            steps {
                sh 'npm -v'
            }
        }
    }
}