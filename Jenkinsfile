pipeline {
    
    agent any 
    
    environment {
        IMAGE_TAG = "${BUILD_NUMBER}"
    }
    
    stages {
        
        stage('Checkout') {
            steps {
                git credentialsId: 'your-git-credentials-id', 
                    url: 'https://github.com/ahamedmareerlive/python-jenkins-argocd-k8s.git', 
                    branch: 'main'
            }
        }

        stage('Build Docker') {
            steps {
                script {
                    sh '''
                    echo 'Build Docker Image'
                    docker build -t mareerahamed/cicde2e:${BUILD_NUMBER} .
                    '''
                }
            }
        }

        stage('Push the artifacts') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials-id', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                        sh '''
                        echo "Logging in to Docker Hub..."
                        echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin
                        echo "Pushing Docker Image..."
                        docker push mareerahamed/cicde2e:${BUILD_NUMBER}
                        '''
                    }
                }
            }
        }
        
        stage('Checkout K8S manifest SCM') {
            steps {
                git credentialsId: 'github-credentials', 
                    url: 'https://github.com/ahamedmareerlive/deploy.git', 
                    branch: 'main'
            }
        }
        
        stage('Update K8S manifest & push to Repo') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'github-credentials', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                        sh '''
                        cat deploy/deploy.yaml
                        sed -i "s|image: mareerahamed/cicde2e:.*|image: mareerahamed/cicde2e:${BUILD_NUMBER}|g" deploy/deploy.yaml
                        cat deploy/deploy.yaml
                        git config --global user.email "jenkins@example.com"
                        git config --global user.name "Jenkins"
                        git add deploy/deploy.yaml
                        git commit -m 'Updated the deploy.yaml | Jenkins Pipeline'
                        git push https://$GIT_USERNAME:$GIT_PASSWORD@github.com/ahamedmareerlive/deploy.git HEAD:main
                        '''                        
                    }
                }
            }
        }
    }
}
