pipeline {
    environment {
        DOCKER_ID = "abeldevops1"
        DOCKER_IMAGE = "datascientestapi"
        DOCKER_TAG = "v.${BUILD_ID}.0"
        DOCKER_PASS = credentials("DOCKER_HUB_SECRET")
    }

    agent any

    stages {
        stage('Cleanup docker containers and images') {
            steps {
                script {
                    sh'''    
                    echo $(docker ps -q)
                    if [[ $(docker ps -q) ]]; then
                        echo "Running containers found. Cleaning up..."
                        docker stop $(docker ps -aq)
                        docker rm $(docker ps -aq)
                        docker rmi -f $(docker images -q)
                    else
                        echo "No running containers found."
                    fi  
                    '''
                }
            }
        }

        stage('Docker image build') {
            steps {
                script {
                    sh 'pwd'
                    sh 'docker-compose -f /var/lib/jenkins/workspace/git/local-test/docker-compose.yml build'
                    sh 'sleep 6'
                }
            }
        }

        stage('Docker image up') {
            steps {
                script {
                    sh 'docker-compose -f /var/lib/jenkins/workspace/git/local-test/docker-compose.yml up -d'
                    sh 'sleep 10'
                }
            }
        }
        
        stage('Image test') {
            steps {
                script {
                    sh 'curl http://0.0.0.0:5000'
                }
            }
        } 

        stage('Build and tag docker image for dockerhub') {
            steps {
                script {
                    sh '''
                    docker build -t $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG .
                    sleep 6
                    '''
                }
            }
        }

        stage('Docker Push') { 
            steps {
                script {
                    withCredentials([string(credentialsId: 'DOCKER_HUB_SECRET', variable: 'DOCKER_PASS')]) {
                        sh '''
                        docker login -u $DOCKER_ID -p $DOCKER_PASS
                        docker push $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG
                        '''
                    }
                }
            }
        }

        stage('Dev deployment') {
            steps {
                script {
                    sh '''
                    echo "Installation Ingress-controller Nginx"
                    helm upgrade --install ingress-nginx ingress-nginx \
                    --repo https://kubernetes.github.io/ingress-nginx \
                    --namespace ingress-nginx --create-namespace     
                    sleep 10

                    echo "Installation Cert-Manager"
                    helm upgrade --install cert-manager cert-manager \
                    --repo https://charts.jetstack.io \
                    --create-namespace --namespace cert-manager \
                    --set installCRDs=true
                    sleep 10
                    
                    echo "Installation stack Prometheus-Grafana"
                    helm upgrade --install kube-prometheus-stack kube-prometheus-stack \
                    --namespace kube-prometheus-stack --create-namespace \
                    --repo https://prometheus-community.github.io/helm-charts

                    echo "Installation Projet Devops 2023"
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" myapp1/values.yaml
                    helm upgrade --install myapp-release-dev myapp1/ --values myapp1/values.yaml -f myapp1/values-dev.yaml -n dev --create-namespace
                    kubectl apply -f myapp1/clusterissuer-prod.yaml    
                    sleep 10
                    '''
                }
            }           
        }

        stage('Staging deployment') {
            steps {
                script {
                    sh '''
                    curl -k -i -X  'POST' -H 'Content-Type: application/json' -d '{"id": 1, "name": "toto", "email": "listen@email.com","password": "passwordtoto"}' https://www.examabel.cloudns.ph.
                    if curl -k -i -H 'accept: application/json' https://www.examabel.cloudns.ph/users | grep -qF "toto"; then
                        echo "La chaîne 'toto' a été trouvée dans la réponse."
                    else
                        echo "La chaîne 'toto' n'a pas été trouvée dans la réponse."
                    fi
                    '''
                }
            }
        }
        
        stage('Production deployment') {
            steps {
                timeout(time: 15, unit: "MINUTES") {
                    input message: 'Do you want to deploy in production ?', ok: 'Yes'
                }
                script {
                    sh '''
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" myapp1/values.yaml     
                    helm upgrade --install myapp-release-prod myapp1/ --values myapp1/values.yaml -f myapp1/values-prod.yaml -n prod --create-namespace
                    kubectl apply -f myapp1/ingress-grafana.yaml
                    kubectl apply -f myapp1/clusterissuer-prod.yaml
                    '''
                }
            }
        }
    }
    
    post {
        success {
            script {
                echo "propre"
            }
        }
        
        failure {
            script {
                echo "pas propre"
            }
        }
    }
}
