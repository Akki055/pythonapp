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

        stage('Checkout K8S Manifests') {
            steps {
                git credentialsId: 'git-cred',
                    url: 'https://github.com/Akki055/pythonapp.git',
                    branch: 'master'
            }
        }

        stage('Update K8S Manifest') {
            steps {
                script {
                    // Read the deploy.yaml file
                    def deployYaml = readYaml file: 'deploy/deploy.yaml'
                    
                    // Update the image tag in the deployment spec
                    deployYaml.spec.template.spec.containers[0].image = "akki058/cicd-e2e:${BUILD_NUMBER}"
                    
                    // Write the updated YAML back to the file
                    writeYaml file: 'deploy/deploy.yaml', data: deployYaml
                    
                    // Print the updated YAML for verification
                    sh "cat deploy/deploy.yaml"
                }
            }
        }

        stage('Commit and Push Changes') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'git-cred', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                        sh '''
                        cd deploy
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
