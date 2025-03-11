pipeline {
    agent any

    environment {
        GIT_REPO_URL = 'https://github.com/k1networth/docker.git'
        GIT_BRANCH = 'main'
        DOCKER_IMAGE_NGINX = "k1networth/my-nginx:finally"
        DOCKER_IMAGE_APACHE = "k1networth/my-apache:finally"
        UBUNTU_USER = "yuuarai"
        UBUNTU_IP = "89.169.145.246"
    }

    stages {
        stage('Delete workspace before build starts') {
            steps {
                deleteDir()
            }    
        }
        
        stage('Clone Repository') {
            steps {
                git branch: "${GIT_BRANCH}", url: "${GIT_REPO_URL}"
            }
        }

        stage('Build images') {
            steps {
                sh 'docker image prune -af'

                sh 'docker build -t nginx ./nginx'
                sh 'docker tag nginx ${DOCKER_IMAGE_NGINX}'
                    
                sh 'docker build -t apache ./apache'
                sh 'docker tag apache ${DOCKER_IMAGE_APACHE}'
            }
        }

        stage('Push Images') {
            steps {
                withDockerRegistry(credentialsId: 'dockerhub-cred-k1networth', url: 'https://index.docker.io/v1/') {
                    sh 'docker push ${DOCKER_IMAGE_NGINX}'
                    
                    sh 'docker push ${DOCKER_IMAGE_APACHE}'
                }
            }
        }
        
        stage('Deploy to instance') {
            steps {
                sshagent(credentials: ['ssh-ubuntu']) {
                    script {
                        sh '''
                        ssh -o StrictHostKeyChecking=no ${UBUNTU_USER}@${UBUNTU_IP} <<EOF
                        
                        docker stop apache || true
                        docker rm apache || true
                        
                        docker stop nginx || true
                        docker rm nginx || true

                        docker network create --driver=bridge --subnet=192.168.1.0/24 network || true

                        docker run -d -p 8080:8080 --name apache --network network ${DOCKER_IMAGE_APACHE}
                        docker run -d -p 80:80 --name nginx --network network ${DOCKER_IMAGE_NGINX}

                        docker image prune -f
                        '''
                    }
                }
            }
        }
    }
}
