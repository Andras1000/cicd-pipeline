@Library('my-shared-library') _

pipeline {
    agent any
    environment {
        registryNamespace = "m3rs"
        repositoryName = "lab-3-cicd-pipeline"
        imageName = "${registryNamespace}/${repositoryName}_${BRANCH_NAME}"
        registryCredential = 'docker_hub_pw'
        dockerImage = ''
        imageReference = ''
        postJobName = ''
    }
    tools {
        nodejs 'node'
    }
    stages {
        stage ('Build') {
            steps {
                sh '''
                   npm install
                   npm build
                   '''
            }
        }
        stage ('Test') {
            steps {
                sh 'npm test'
            }
        }
        stage ('Docker build') {
            steps {
                script {
                    imageReference = "${imageName}:${BUILD_NUMBER}"
                    dockerImage = docker.build imageReference
                }
            }
        }
        stage ('Vulnerability Scan') {
            steps {
                script {
                    def vulnerabilities = sh(script: "trivy image --exit-code 0 --severity HIGH,MEDIUM,LOW --no-progress ${imageReference}", returnStdout: true).trim()
                    echo "Vulnerability report:\n${vulnerabilities}"
                }
            }
        }
        stage ('Deploy (push image)') {
            steps {
                script {
                    docker.withRegistry( '', registryCredential ) {
                        dockerImage.push()
                    }
                }
            }
        }
        stage ('Clean up') {
            steps {
                script {
                    mySharedLibrary.removeImage(imageReference)
                }
            }
        }
    }
    post ('Start deploy pipeline') {
        success {
            script {
                def branchName = env.BRANCH_NAME
                if (branchName == 'dev') {
                    postJobName = 'Deploy_to_dev'
                } else if (branchName == 'main') {
                    postJobName = 'Deploy_to_main'
                }
            }
            build job: postJobName, parameters: [string(name: 'IMAGE_REFERENCE', value: imageReference)]
        }
    }
}