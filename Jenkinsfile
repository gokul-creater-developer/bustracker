\
        pipeline {
            agent any

            environment {
                DOCKERHUB_USER = 'gokulnath2002'
                APP_NAME = 'flask-cicd-app'
                EC2_HOST = '' // set your EC2 public IP or hostname in Jenkins job configuration or credentials
            }

            stages {
                stage('Checkout') {
                    steps {
                        checkout scm
                    }
                }

                stage('Install Dependencies') {
                    steps {
                        sh 'python3 -m pip install --upgrade pip || true'
                        sh 'pip install -r requirements.txt'
                    }
                }

                stage('Run Tests') {
                    steps {
                        sh 'pytest --maxfail=1 --disable-warnings -q'
                    }
                }

                stage('Build Docker Image') {
                    steps {
                        sh "docker build -t ${env.DOCKERHUB_USER}/${env.APP_NAME}:latest ."
                    }
                }

                stage('Push to DockerHub') {
                    steps {
                        withCredentials([usernamePassword(credentialsId: 'dockerhub_login', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                            sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                            sh "docker push ${env.DOCKERHUB_USER}/${env.APP_NAME}:latest"
                        }
                    }
                }

                stage('Deploy to EC2') {
                    steps {
                        echo 'Deploying to EC2 via SSH (requires ssh-agent credentials in Jenkins)'
                        // This stage assumes you have configured SSH credentials in Jenkins with id 'ec2_ssh'
                        // and have set EC2_HOST environment variable (pipeline job parameter or global env).
                        sshagent (credentials: ['ec2_ssh']) {
                            sh (
                                "ssh -o StrictHostKeyChecking=no ec2-user@${env.EC2_HOST} " +
                                "'docker pull ${env.DOCKERHUB_USER}/${env.APP_NAME}:latest && " +
                                "docker stop flask-app || true && docker rm flask-app || true && " +
                                "docker run -d -p 5000:5000 --name flask-app ${env.DOCKERHUB_USER}/${env.APP_NAME}:latest'"
                            )
                        }
                    }
                }
            }

            post {
                success {
                    echo '✅ CI/CD pipeline completed successfully!'
                }
                failure {
                    echo '❌ Pipeline failed. Check logs.'
                }
            }
        }
