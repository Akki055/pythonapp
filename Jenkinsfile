pipeline {
    
    agent any 
    
    environment {
        IMAGE_TAG = "${BUILD_NUMBER}"
    }
    
    stages {
        
        stage('Checkout'){
           steps {
                git credentialsId: 'git-cred', 
                url: 'https://github.com/Akki055/pythonapp.git',
                branch: 'master'
           }
        }

        stage('Build Docker'){
            steps{
                script{
                    withDockerRegistry(credentialsId: 'doc-cred') {
                     sh '''
                     echo 'Buid Docker Image'
                     docker build -t akki058/cicd-e2e:${BUILD_NUMBER} .
                     '''
                  }
                }
            }
        }

        stage('Push the artifacts'){
           steps{
                script{
                    withDockerRegistry(credentialsId: 'doc-cred') {
                     sh '''
                     echo 'Push to Repo'
                     docker push akki058/cicd-e2e:${BUILD_NUMBER}
                     '''
                    }
                }
            }
        }
        
        stage('Checkout K8S manifest SCM'){
            steps {
                git credentialsId: 'git-cred', 
                url: 'https://github.com/Akki055/pythonapp.git',
                branch: 'master'
            }
        }
        
        stage('Update K8S manifest & push to Repo'){
            steps {
                script{
                    withCredentials([usernamePassword(credentialsId: 'git-cred', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                        sh '''
                        cd deploy
                        cat deploy.yaml
                        sed -i '' "s/32/${BUILD_NUMBER}/g" deploy.yaml
                        cat deploy.yaml
                        git add deploy.yaml
                        git commit -m 'Updated the deploy yaml | Jenkins Pipeline'
                        git remote -v
                        git push https://github.com/Akki055/pythonapp.git HEAD:master
                        ''' 
                    }
                }
            }
        }
    }
}
