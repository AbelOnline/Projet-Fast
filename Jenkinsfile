pipeline {
    agent any
    environment {
        DOCKER_ID = "abeldevops1"
        DOCKER_IMAGE = "datascientestapi"
        DOCKER_TAG = "v.${BUILD_ID}.0"
        DOCKER_HUB_SECRET = credentials("DOCKER_HUB_SECRET")
    }
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
        stage('Build and tag docker image for Docker Hub') {
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
        stage('Staging Deployment') {
            steps {
                script {
                    try {
                        sh '''
                        curl -k -i -X POST -H 'Content-Type: application/json' -d '{"id": 1, "name": "toto", "email": "toto@email.com","password": "passwordtoto"}' https://www.devops-youss.cloudns.ph
                        '''
                        def response = sh(script: 'curl -k -i -H "accept: application/json" https://www.devops-youss.cloudns.ph/users', returnStatus: true)
                        
                        if (response == 0) {
                            echo "The string 'toto' was found in the response."
                        } else {
                            echo "The string 'toto' was not found in the response."
                            error "Staging deployment failed."
                        }
                    } catch (Exception e) {
                        currentBuild.result = 'FAILURE'
                        error "An error occurred during staging deployment: ${e.message}"
                    }
                }
            }
        }
        stage('Production Deployment') {
            steps {
                timeout(time: 15, unit: 'MINUTES') {
                    input message: 'Do you want to deploy in production?', ok: 'Yes'
                }

                script {
                    try {
                        sh '''
                        sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" myapp1/values.yaml
                        helm upgrade --install myapp-release-prod myapp1/ --values myapp1/values.yaml -f myapp1/values-prod.yaml -n prod --create-namespace
                        kubectl apply -f myapp1/ingress-grafana.yaml
                        kubectl apply -f myapp1/clusterissuer-prod.yaml
                        '''
                    } catch (Exception e) {
                        currentBuild.result = 'FAILURE'
                        error "An error occurred during production deployment: ${e.message}"
                    }
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
