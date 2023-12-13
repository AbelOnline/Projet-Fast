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
                    sh 'aws configure set aws_access_key_id AKIA2MXZW63VD7RDXRBQ --profile Abel'
                    sh 'aws configure set aws_secret_access_key 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 --profile Abel'
                    sh 'aws configure set region eu-west-3 --profile Abel'
                    sh 'aws eks update-kubeconfig --name eks --region eu-west-3 --profile Abel'
                    sh 'helm delete ingress-nginx --purge || true'
                    sh 'helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx'
                    sh 'helm repo update'
                    sh 'helm install ingress-nginx ingress-nginx/ingress-nginx --namespace ingress-nginx --create-namespace'
                }
            }
        }
    }

    post {
        success {
            script {
                echo "Deployment was successful."
            }
        }
        failure {
            script {
                echo "Deployment failed."
            }
        }
    }
}
