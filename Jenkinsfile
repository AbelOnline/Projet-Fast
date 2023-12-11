pipeline {
    environment { // Declaration of environment variables
    DOCKER_ID = "abeldevops1" // replace this with your docker-id
    DOCKER_IMAGE = "datascientestapi"
    DOCKER_TAG = "v.${BUILD_ID}.0" // we will tag our images with the current build in order to increment the value by 1 with each new build
    DOCKER_PASS = nsVKL9X8g6$+$k%
    //AWS_ACCESS_KEY_ID = credentials('AWS_ACCESS_KEY_ID')
   // AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY') 
    //AWS_DEFAULT_REGION = "eu-west-3" //
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

        // Build docker image
        stage('Docker image build') {
            steps {
                script {
                     
                        // sh 'cd local-test'
                        sh 'pwd'
                        sh 'docker-compose -f /var/lib/jenkins/workspace/git/local-test/docker-compose.yml build'
                        sh 'sleep 6'
                    
                }
            }
        }
        // Run the docker image
        stage('Docker image up') {
                steps {
                script {
                        sh 'docker-compose -f /var/lib/jenkins/workspace/git/local-test/docker-compose.yml up -d'
                        // sh ' cd .. '
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
            withCredentials([string(credentialsId: 'DOCKER_HUB_PASS', variable: 'DOCKER_PASS')]) {
                sh '''
                echo "$DOCKER_PASS" | docker login -u abeldevops1 --password-stdin
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
                    fi'''
                    
                }
            }
        }
        
        stage('Production deployment') {
            steps {
                // Create an Approval Button with a timeout of 15minutes.
                // this require a manuel validation in order to deploy on production environment
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
    
    // }
}
}
