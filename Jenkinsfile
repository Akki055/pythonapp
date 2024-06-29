pipeline {
    agent any

    environment {
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout Source') {
            steps {
                git credentialsId: 'git-cred',
                    url: 'https://github.com/Akki055/pythonapp.git',
                    branch: 'master'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'doc-cred') {
                        sh '''
                        echo 'Building Docker Image'
                        docker build -t akki058/cicd-e2e:${BUILD_NUMBER} .
                        '''
                    }
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'doc-cred') {
                        sh '''
                        echo 'Pushing Docker Image to Repository'
                        docker push akki058/cicd-e2e:${BUILD_NUMBER}
                        '''
                    }
                }
            }
        }

        stage('Update K8S Manifest') {
            steps {
                script {
                    // Navigate to the deploy directory
                    dir('deploy') {
                        // Use sed to update the image tag in deploy.yaml
                        sh '''
                        echo "Updating image tag in deploy.yaml to akki058/cicd-e2e:${BUILD_NUMBER}"
                        
                        # Print the current content for debugging
                        echo "Current deploy.yaml content:"
                        cat deploy.yaml
                        
                        # Use sed to find and replace the image tag in the deploy.yaml
                        sed -i 's|image: akki058/cicd-e2e:.*|image: akki058/cicd-e2e:${BUILD_NUMBER}|' deploy.yaml
                        
                        # Print the updated content for verification
                        echo "Updated deploy.yaml content:"
                        cat deploy.yaml
                        '''
                    }
                }
            }
        }

        stage('Commit and Push Changes') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'git-cred', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                        // Navigate to the deploy directory for git operations
                        dir('deploy') {
                            sh '''
                            git add deploy.yaml
                            git commit -m "Updated deploy.yaml with new image tag ${BUILD_NUMBER}"
                            
                            # Using HTTPS URL with credentials to push changes
                            git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/Akki055/pythonapp.git HEAD:master
                            '''
                        }
                    }
                }
            }
        }
    }

    post {
        failure {
            echo "Pipeline failed. Check the logs for errors."
        }
    }
}
