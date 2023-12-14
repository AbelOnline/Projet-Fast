pipeline {
    environment {
        DOCKER_ID = "abeldevops1"
        DOCKER_IMAGE = "datascientestapi"
        DOCKER_TAG = "v.${BUILD_ID}.0"
        AWS_DEFAULT_REGION = "eu-west-3"
        DOCKER_HUB_SECRET = credentials("DOCKER_HUB_SECRET")
    }
    agent any
    stages {
        stage('Cleanup docker containers and images') {
            steps {
                script {
                    sh '''    
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
        stage('Docker Tag and Push') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'DOCKER_HUB_SECRET', variable: 'DOCKER_HUB_SECRET')]) {
                        def dockerId = env.DOCKER_ID
                        def dockerImage = env.DOCKER_IMAGE
                        def dockerTag = env.DOCKER_TAG
                        def dockerHubSecret = env.DOCKER_HUB_SECRET
                        sh """
                        echo '$dockerHubSecret' | docker login -u $dockerId --password-stdin
                        
                        docker tag $dockerId/$dockerImage:$dockerTag $dockerId/$dockerImage:NouveauTag
                        docker push $dockerId/$dockerImage:NouveauTag
                        """
                    }
                }
            }
        }
        stage('Dev deployment') {
            steps {
                script {
                    sh 'aws configure set aws_access_key_id AKIA2MXZW63VMASNZDGJ --profile Abel'
                    sh 'aws configure set aws_secret_access_key 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 --profile Abel'
                    sh 'aws configure set region eu-west-3 --profile Abel'
                    
                    sh 'aws eks update-kubeconfig --name eks --region eu-west-3 --profile Abel'
                    
                    echo "Installation Projet Devops 2023"
                    echo "Installation stack Prometheus-Grafana"
                    helm upgrade --install kube-prometheus-stack kube-prometheus-stack \
                    --namespace kube-prometheus-stack --create-namespace \
                    --repo https://prometheus-community.github.io/helm-charts
                    
                    sh '''sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" myapp1/values.yaml
                          helm upgrade --install myapp-release-dev myapp1/ --values myapp1/values.yaml \
                          -f myapp1/values-dev.yaml -n dev --create-namespace
                          sleep 10
                          '''
                    
                    // Reste de votre code de déploiement
                }
            }
        }
        stage('Staging deployment') {
            steps {
                script {
                    sh '''
                    curl -k -i -X 'POST' -H 'Content-Type: application/json' \
                    -d '{"id": 1, "name": "toto", "email": "toto@email.com","password": "passwordtoto"}' \
                    https://www. examabel.cloudns.ph
                    
                    if curl -k -i -H 'accept: application/json' https://www. examabel.cloudns.ph/users | grep -qF "toto"; then
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
                    sed -i "s+tag.*+tag: \${DOCKER_TAG}+g" myapp1/values.yaml
                    helm upgrade --install myapp-release-prod myapp1/ --values myapp1/values.yaml \
                    -f myapp1/values-prod.yaml -n prod --create-namespace
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
