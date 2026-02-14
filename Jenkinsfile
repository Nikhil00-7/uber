pipeline {
    agent any 

    tools {
        nodejs "node22"
    }

    environment {
        DOCKER_USER = "docdon0007"
        TAG = "03"
        
        USER_IMAGE = "${DOCKER_USER}/user:${TAG}"
        RIDER_IMAGE = "${DOCKER_USER}/rider:${TAG}"
        CAPTAIN_IMAGE = "${DOCKER_USER}/captain:${TAG}"
        GATEWAY_IMAGE = "${DOCKER_USER}/apigateway:${TAG}"

        PATH = "/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin"
    }

    stages {

        stage("Checkout Code") {
            steps {
                git branch: 'main', url: 'https://github.com/Nikhil00-7/uber.git'
            }
        }

        stage("Install Dependencies") {
            steps {
                sh '''
                    npm -v
                    node -v
                    npm install
                '''
            }
        }

        stage("Docker Login & Push") {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-login',
                    usernameVariable: 'DOCKER_USERNAME',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo $DOCKER_PASS | docker login -u $DOCKER_USERNAME --password-stdin

                        docker build -t $USER_IMAGE ./micro-services/user
                        docker build -t $RIDER_IMAGE ./micro-services/ride
                        docker build -t $CAPTAIN_IMAGE ./micro-services/captain
                        docker build -t $GATEWAY_IMAGE ./micro-services/gateway

                        docker push $USER_IMAGE
                        docker push $RIDER_IMAGE
                        docker push $CAPTAIN_IMAGE
                        docker push $GATEWAY_IMAGE
                    '''
                }
            }
        }

        stage("Push Kubernetes YAMLs") {
            steps {
                sshagent(['k8s-master-ssh']) {
                    sh '''
                        scp -r micro-services/k8s ubuntu@192.168.2.20:/home/ubuntu/
                    '''
                }
            }
        }

        stage("Deploy to Kubernetes") {
            steps {
                sshagent(['k8s-master-ssh']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no ubuntu@192.168.2.20 "
                            kubectl get ns uber || kubectl create ns uber
                            kubectl apply -f /home/ubuntu/k8s --recursive -n uber
                        "
                    '''
                }
            }
        }

    }  // ✅ closes stages

    post {
        success {
            echo "Deployment finished successfully!"
        }
        failure {
            echo "Deployment failed."
        }
    }

}  // ✅ closes pipeline
