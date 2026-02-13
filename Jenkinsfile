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

        USER_DEPLOYMENT = "micro-services/k8s/user-service/user-deployment.yaml"
        RIDER_DEPLOYMENT = "micro-services/k8s/ride-service/ride-deployment.yaml"
        CAPTAIN_DEPLOYMENT = "micro-services/k8s/captain-service/captain-deployment.yaml"
        
        USER_SERVICE = "micro-services/k8s/user-service/user-service.yaml"
        RIDER_SERVICE = "micro-services/k8s/ride-service/ride-service.yaml"
        CAPTAIN_SERVICE = "micro-services/k8s/captain-service/captain-service.yaml"
        
        USER_CONFIG_MAP = "micro-services/k8s/user-service/user-configMap.yaml"
        USER_SECRET = "micro-services/k8s/user-service/user-secret.yaml"
        RIDER_CONFIG_MAP = "micro-services/k8s/ride-service/rider-configMap.yaml"
        RIDER_SECRET = "micro-services/k8s/ride-service/rider-secret.yaml"

        GATEWAY_DEPLOYMENT = "micro-services/k8s/gateway-service/gate-way-deployment.yaml"
        GATEWAY_SERVICE = "micro-services/k8s/gateway-service/gate-way-service.yaml"
        
        CAPTAIN_SERVICE_HPA = "micro-services/k8s/captain-service/captain-service-hpa.yaml"
        CAPTAIN_DB_STATEFULSET = "micro-services/k8s/captain-service/captain-db-statefullset.yaml"
        CAPTAIN_STATEFULSET_SERVICE = "micro-services/k8s/captain-service/captain-stateful-set-service.yaml"
        
        RIDE_STATEFULSET = "micro-services/k8s/ride-service/ride-statefull-set.yaml"
        RIDE_STATEFULSET_SERVICE = "micro-services/k8s/ride-service/ride-statefull-set-service.yaml"
        RIDE_SERVICE_HPA = "micro-services/k8s/ride-service/ride-service-hpa.yaml"
        
        USER_STATEFULSET_SERVICE = "micro-services/k8s/user-service/user-statefull-set-service.yaml"
        USER_STATEFULSET = "micro-services/k8s/user-service/user-statefull-set.yaml"
        USER_SERVICE_HPA = "micro-services/k8s/user-service/user-service-hpa.yaml"

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
                          docker build -t $USER_IMAGE ./micro-services/user
                docker build -t $RIDER_IMAGE ./micro-services/ride
                docker build -t $CAPTAIN_IMAGE ./micro-services/captain
                docker build -t $GATEWAY_IMAGE ./micro-services/gateway
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
            script {
                def files = [
                    USER_DEPLOYMENT,
                    RIDER_DEPLOYMENT,
                    CAPTAIN_DEPLOYMENT,
                    USER_SERVICE,
                    RIDER_SERVICE,
                    CAPTAIN_SERVICE,
                    USER_CONFIG_MAP,
                    USER_SECRET,
                    RIDER_CONFIG_MAP,
                    RIDER_SECRET,
                    GATEWAY_DEPLOYMENT,
                    GATEWAY_SERVICE,
                    CAPTAIN_SERVICE_HPA,
                    CAPTAIN_DB_STATEFULSET,
                    CAPTAIN_STATEFULSET_SERVICE,
                    RIDE_STATEFULSET,
                    RIDE_STATEFULSET_SERVICE,
                    RIDE_SERVICE_HPA,
                    USER_STATEFULSET_SERVICE,
                    USER_STATEFULSET,
                    USER_SERVICE_HPA
                ]

                for (file in files) {
                    def filename = file.tokenize('/').last()
                    sh """
                        ssh -o StrictHostKeyChecking=no ubuntu@192.168.2.20 \
                        "kubectl apply -f /home/ubuntu/${filename} -n uber"
                    """
                }
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
