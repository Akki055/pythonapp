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
                        // Print out the current directory for debugging
                        sh 'echo "Current directory: $(pwd)"'
                        sh 'ls -l'

                        // Read the deploy.yaml file
                        def deployYaml = readYaml file: 'deploy.yaml'
                        
                        // Print the existing YAML structure for debugging
                        echo 'Current deploy.yaml content:'
                        echo deployYaml.toString()
                        
                        // Update the image tag in the deployment spec
                        deployYaml.spec.template.spec.containers[0].image = "akki058/cicd-e2e:${BUILD_NUMBER}"
                        
                        // Write the updated YAML back to the file
                        writeYaml file: 'deploy.yaml', data: deployYaml
                        
                        // Print the updated YAML for verification
                        sh 'echo "Updated deploy.yaml content:"'
                        sh 'cat deploy.yaml'
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
