pipeline {
    agent any 

    tools {
        nodejs "node22"
    }

    environment {
        DOCKER_USER = "docdon0007"
        TAG = "01"
        
        USER_IMAGE = "${DOCKER_USER}/user:${TAG}"
        RIDER_IMAGE = "${DOCKER_USER}/rider:${TAG}"
        CAPTAIN_IMAGE = "${DOCKER_USER}/captain:${TAG}"
        GATEWAY_IMAGE = "${DOCKER_USER}/apigateway:${TAG}"

        USER_DEPLOYMENT = "user-deployment.yaml"
        RIDER_DEPLOYMENT = "ride-deployment.yaml"
        CAPTAIN_DEPLOYMENT = "captain-deployment.yaml"
        
        USER_SERVICE = "user-service.yaml"
        RIDER_SERVICE = "ride-service.yaml"
        CAPTAIN_SERVICE = "captain-service.yaml"
        
        USER_CONFIG_MAP = "user-configMap.yaml"
        USER_SECRET = "user-secret.yaml"
        RIDER_CONFIG_MAP = "rider-configMap.yaml"
        RIDER_SECRET = "rider-secret.yaml"

        GATEWAY_DEPLOYMENT = "gate-way-deployment.yaml"
        GATEWAY_SERVICE = "gate-way-service.yaml"
        
        CAPTAIN_SERVICE_HPA = "captain-service-hpa.yaml"
        CAPTAIN_DB_STATEFULSET = "captain-db-statefullset.yaml"
        CAPTAIN_STATEFULSET_SERVICE = "captain-stateful-set-service.yaml"
        
        RIDE_STATEFULSET = "ride-statefull-set.yaml"
        RIDE_STATEFULSET_SERVICE = "ride-statefull-set-service.yaml"
        RIDE_SERVICE_HPA = "ride-service-hpa.yaml"
        
        USER_STATEFULSET_SERVICE = "user-statefull-set-service.yaml"
        USER_STATEFULSET = "user-statefull-set.yaml"
        USER_SERVICE_HPA = "user-service-hpa.yaml"

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
                sh 'npm -v'
                sh 'node -v'
                sh 'npm install'
            }
        }

        stage("Docker Login & Push") {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-login', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker build -t $USER_IMAGE .
                        docker build -t $RIDER_IMAGE .
                        docker build -t $CAPTAIN_IMAGE .
                        docker build -t $GATEWAY_IMAGE .
                        docker push $USER_IMAGE
                        docker push $RIDER_IMAGE
                        docker push $CAPTAIN_IMAGE
                        docker push $GATEWAY_IMAGE
                    """
                }
            }
        }

        stage("Push Kubernetes YAMLs") {
            steps {
                sshagent(['k8s-master-ssh']) {
                    sh """
                        scp ${USER_DEPLOYMENT} ubuntu@192.168.2.20:/home/ubuntu/
                        scp ${RIDER_DEPLOYMENT} ubuntu@192.168.2.20:/home/ubuntu/
                        scp ${CAPTAIN_DEPLOYMENT} ubuntu@192.168.2.20:/home/ubuntu/
                        scp ${USER_SERVICE} ubuntu@192.168.2.20:/home/ubuntu/
                        scp ${RIDER_SERVICE} ubuntu@192.168.2.20:/home/ubuntu/
                        scp ${CAPTAIN_SERVICE} ubuntu@192.168.2.20:/home/ubuntu/
                        scp ${USER_CONFIG_MAP} ubuntu@192.168.2.20:/home/ubuntu/
                        scp ${USER_SECRET} ubuntu@192.168.2.20:/home/ubuntu/
                        scp ${RIDER_CONFIG_MAP} ubuntu@192.168.2.20:/home/ubuntu/
                        scp ${RIDER_SECRET} ubuntu@192.168.2.20:/home/ubuntu/
                        scp ${GATEWAY_DEPLOYMENT} ubuntu@192.168.2.20:/home/ubuntu/
                        scp ${GATEWAY_SERVICE} ubuntu@192.168.2.20:/home/ubuntu/
                        scp ${CAPTAIN_SERVICE_HPA} ubuntu@192.168.2.20:/home/ubuntu/
                        scp ${CAPTAIN_DB_STATEFULSET} ubuntu@192.168.2.20:/home/ubuntu/
                        scp ${CAPTAIN_STATEFULSET_SERVICE} ubuntu@192.168.2.20:/home/ubuntu/
                        scp ${RIDE_STATEFULSET} ubuntu@192.168.2.20:/home/ubuntu/
                        scp ${RIDE_STATEFULSET_SERVICE} ubuntu@192.168.2.20:/home/ubuntu/
                        scp ${RIDE_SERVICE_HPA} ubuntu@192.168.2.20:/home/ubuntu/
                        scp ${USER_STATEFULSET_SERVICE} ubuntu@192.168.2.20:/home/ubuntu/
                        scp ${USER_STATEFULSET} ubuntu@192.168.2.20:/home/ubuntu/
                        scp ${USER_SERVICE_HPA} ubuntu@192.168.2.20:/home/ubuntu/
                    """
                }
            }
        }

        stage("Deploy to Kubernetes") {
            steps {
                sshagent(['k8s-master-ssh']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no ubuntu@192.168.2.20 "kubectl get ns uber || kubectl create ns uber"

                        FILES=(
                          ${USER_DEPLOYMENT}
                          ${RIDER_DEPLOYMENT}
                          ${CAPTAIN_DEPLOYMENT}
                          ${USER_SERVICE}
                          ${RIDER_SERVICE}
                          ${CAPTAIN_SERVICE}
                          ${USER_CONFIG_MAP}
                          ${USER_SECRET}
                          ${RIDER_CONFIG_MAP}
                          ${RIDER_SECRET}
                          ${GATEWAY_DEPLOYMENT}
                          ${GATEWAY_SERVICE}
                          ${CAPTAIN_SERVICE_HPA}
                          ${CAPTAIN_DB_STATEFULSET}
                          ${CAPTAIN_STATEFULSET_SERVICE}
                          ${RIDE_STATEFULSET}
                          ${RIDE_STATEFULSET_SERVICE}
                          ${RIDE_SERVICE_HPA}
                          ${USER_STATEFULSET_SERVICE}
                          ${USER_STATEFULSET}
                          ${USER_SERVICE_HPA}
                        )

                        for file in "${FILES[@]}"; do
                          ssh -o StrictHostKeyChecking=no ubuntu@192.168.2.20 "kubectl apply -f /home/ubuntu/$file --namespace=uber"
                        done
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "Deployment finished successfully!"
        }
        failure {
            echo "Deployment failed."
        }
    }
}
