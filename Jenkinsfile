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
       stage('Docker Tag and Push') {
    steps {
        script {
            withCredentials([string(credentialsId: 'DOCKER_HUB_SECRET', variable: 'DOCKER_HUB_SECRET')]) {
                // Assurez-vous que DOCKER_ID, DOCKER_IMAGE, DOCKER_TAG sont définis en tant que variables d'environnement dans votre pipeline Jenkins.
                // Vous pouvez les définir dans votre pipeline ou les obtenir à partir de votre code source ou d'autres sources.
                def dockerId = env.DOCKER_ID
                def dockerImage = env.DOCKER_IMAGE
                def dockerTag = env.DOCKER_TAG
                def dockerHubSecret = env.DOCKER_HUB_SECRET
                // Utilisez l'option --password-stdin pour éviter de passer le mot de passe directement
                sh """
                echo '$dockerHubSecret' | docker login -u $dockerId --password-stdin
                
                # Tag l'image avec le nouveau tag
                docker tag $dockerId/$dockerImage:$dockerTag $dockerId/$dockerImage:NouveauTag
                # Poussez l'image taggée
                docker push $dockerId/$dockerImage:NouveauTag
                """
            }
        }
    }
}
        stage('Dev deployment') {
    steps {
        script {
            // Configurer le profil AWS "Abel"
            sh 'aws configure set aws_access_key_id AKIA2MXZW63VD7RDXRBQ --profile Abel'
            sh 'aws configure set aws_secret_access_key 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 --profile Abel'
            sh 'aws configure set region eu-west-3 --profile Abel'
            // Mise à jour de kubeconfig pour le cluster EKS
            sh 'aws eks update-kubeconfig --name eks --region eu-west-3 --profile Abel'
            echo "Installation Ingress-controller Nginx"
            helm upgrade --install ingress-nginx ingress-nginx \
            --repo https://kubernetes.github.io/ingress-nginx \
            --namespace ingress-nginx --create-namespace
            sleep 10

            // Reste de votre code de déploiement
            // Reste de votre code de déploiement peut être ajouté ici
            // Assurez-vous de respecter la syntaxe et les étapes appropriées pour votre application.
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
