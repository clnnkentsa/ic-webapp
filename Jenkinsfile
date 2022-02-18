pipeline {
    environment {
        IMAGE_NAME = "ic-webapp"
        APP_CONTAINER_PORT = "8080"
        DOCKERHUB_ID = "choco1992"
        DOCKERHUB_PASSWORD = credentials('dockerhub_password')
        ANSIBLE_IMAGE_AGENT = "registry.gitlab.com/robconnolly/docker-ansible:latest"
    }
    agent none
    stages {
       stage('Build image') {
           agent any
           steps {
              script {
                sh 'docker build --no-cache -f ./sources/app/${DOCKERFILE_NAME} -t ${DOCKERHUB_ID}/$IMAGE_NAME:$IMAGE_TAG ./sources/app'
                
              }
           }
       }


        stage('Scan Dockerfile ${DOCKERFILE_NAME} with  SNYK') {
            /*agent { docker { 
                        image 'franela/dind' 
                        args '-v /var/run/docker.sock:/var/run/docker.sock'
                    } 
            }*/
            
            environment{
                SNYK_TOKEN = credentials('snyk_token')
            }
            steps {
                script{
                    sh '''
#                   apk --no-cache add npm
#                    npm install -g snyk-to-html
                    echo "Starting scan of Dockerfile ${DOCKERFILE_NAME}..."
                    docker run --rm -it --env SNYK_TOKEN -v /var/run/docker.sock:/var/run/docker.sock -v $(pwd):/app snyk/snyk:docker snyk test --docker $DOCKERHUB_ID/$IMAGE_NAME:$IMAGE_TAG --file=./sources/app/${DOCKERFILE_NAME} --json > resultats_${DOCKERFILE_NAME}.json
                    echo `grep 'message' resultats_${DOCKERFILE_NAME}.json`
                    snyk-to-html -i resultats_${DOCKERFILE_NAME}.json -o resultats_${DOCKERFILE_NAME}.html
                    echo "Scan ended"
                    '''
                }
            }
        }

        stage('Scan builded image ${DOCKERHUB_ID}/${IMAGE_NAME}:${IMAGE_TAG} with  SNYK') {
            /*agent { docker { 
                        image 'franela/dind' 
                        args '-v /var/run/docker.sock:/var/run/docker.sock'
                    } 
            }*/
            
            environment{
                SNYK_TOKEN = credentials('snyk_token')
            }
            steps {
                script{
                    sh '''
#                   apk --no-cache add npm
#                    npm install -g snyk-to-html
                    echo "Starting scan of Builded image  ${DOCKERHUB_ID}/$IMAGE_NAME:$IMAGE_TAG..."
                    docker run --rm -it --env SNYK_TOKEN -v /var/run/docker.sock:/var/run/docker.sock -v $(pwd):/app snyk/snyk:docker snyk test --docker $DOCKERHUB_ID/$IMAGE_NAME:$IMAGE_TAG --json > resultats_${DOCKERHUB_ID}/$IMAGE_NAME:$IMAGE_TAG.json
                    echo `grep 'message' resultats_${DOCKERHUB_ID}/$IMAGE_NAME:$IMAGE_TAG.json`
#                   snyk-to-html -i resultats_${DOCKERHUB_ID}/$IMAGE_NAME:$IMAGE_TAG.json -o resultats_${DOCKERHUB_ID}/$IMAGE_NAME:$IMAGE_TAG.html
                    echo "Scan ended"
                    '''
                }
            }
        }

       stage('Run container based on builded image') {
          agent any
          steps {
            script {
              sh '''
                  echo "Cleaning existing container if exist"
                  docker ps -a | grep -i $IMAGE_NAME && docker rm -f ${IMAGE_NAME}
                  docker run --name ${IMAGE_NAME} -d -p $APP_EXPOSED_PORT:$APP_CONTAINER_PORT  ${DOCKERHUB_ID}/$IMAGE_NAME:$IMAGE_TAG
                  sleep 5
              '''
             }
          }
       }
       stage('Test image') {
           agent any
           steps {
              script {
                sh '''
                   curl -I http://${HOST_IP}:${APP_EXPOSED_PORT} | grep -i "200"
                '''
              }
           }
       }
       stage('Clean container') {
          agent any
          steps {
             script {
               sh '''
                   docker stop $IMAGE_NAME
                   docker rm $IMAGE_NAME
               '''
             }
          }
        }

        stage ('Login and Push Image on docker hub') {
          agent any
          steps {
             script {
               sh '''
                   echo $DOCKERHUB_PASSWORD | docker login -u $DOCKERHUB_ID --password-stdin
                   docker push ${DOCKERHUB_ID}/$IMAGE_NAME:$IMAGE_TAG
               '''
             }
          }
        }

    }

}