pipeline {
    environment { // Declaration of environment variables
        DOCKER_ID = "hmatondo"  //DockerHub useraccount
        DOCKER_CAST_IMAGE = "jenkins_devops_exams-cast_service"    // Cast service image
        DOCKER_MOVIES_IMAGE = "jenkins_devops_exams-movie_service"    // Movie service image
        DOCKER_CAST_DB_IMAGE = "postgres:12.1-alpine"    // Cast database image
        DOCKER_MOVIE_DB_IMAGE = "postgres:12.1-alpine"    // Movie database image
        DOCKER_TAG = "v.${BUILD_ID}.0" // Tag our api(s) images with the current build in order to increment the value by 1 with each new build
        HELM_PATH = "/usr/local/bin/helm" // In my case it is important to specify helm path ; if none Jenkins will not find tool
    }
    agent any // Jenkins will be able to select all available agents

    stages {
        stage('Debug Docker Build'){
            steps {
                script {
                    echo "START PIPELINE JENKINS DEVOPS EXAM TEST"
                    echo "DOCKER_ID: $DOCKER_ID"
                    echo "DOCKER_TAG: $DOCKER_TAG"
                    echo "DOCKER_CAST_IMAGE: $DOCKER_CAST_IMAGE"
                    echo "DOCKER_MOVIES_IMAGE: $DOCKER_MOVIES_IMAGE"
                    echo "DOCKER_CAST_DB_IMAGE: $DOCKER_CAST_DB_IMAGE"
                    echo "DOCKER_MOVIE_DB_IMAGE: $DOCKER_MOVIE_DB_IMAGE"
                    echo 'STAGE SUCCESS : Debug Docker Build'
                }
            }
        }

        stage('Docker Compose -> Build and Run'){ // docker build image stage
            steps {
                script {
                sh '''
                  docker rm -f jenkins_devops_exams-cast_db-1
                  docker rm -f jenkins_devops_exams-movie_db-1
                  docker rm -f jenkins_devops_exams-movie_service-1
                  docker rm -f jenkins_devops_exams-cast_service-1
                  docker rm -f jenkins_devops_exams-nginx-1
                  docker compose up -d
                  echo 'STAGE SUCCESS : Docker Compose'
                  sleep 6
                '''
                }
            }
        }

        stage('Docker Tag Images'){ // docker tag image to be able to push them to DockerHub
            steps {
                script {
                sh '''
                  docker ps
                  docker images
                  docker tag jenkins_devops_exams-cast_service $DOCKER_ID/jenkins_devops_exams-cast_service:$DOCKER_TAG
                  docker tag jenkins_devops_exams-movie_service $DOCKER_ID/jenkins_devops_exams-movie_service:$DOCKER_TAG
                  echo 'STAGE SUCCESS : Docker Tag'
                  sleep 4
                '''
                }
            }
        }

        stage('Test Acceptance'){ // we launch the curl command to validate that the container responds to the request
            steps {
                    script {
                    sh '''
                      curl localhost
                      echo "STAGE SUCCESS : Test d'acceptation réussi"
                    '''
                    }
            }

        }

        stage('Docker Push'){ //we pass the built image to our docker hub account
            environment
            {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS") // we retrieve docker password from secret text called docker_hub_pass saved on jenkins
            }

            steps {

                script {
                sh '''
                  docker login -u $DOCKER_ID -p $DOCKER_PASS
                  docker push $DOCKER_ID/$DOCKER_CAST_IMAGE:$DOCKER_TAG
                  docker push $DOCKER_ID/$DOCKER_MOVIES_IMAGE:$DOCKER_TAG
                  echo 'STAGE SUCCESS : Docker Push - Images des applications disponibles sur DockerHub'
                '''
                }
            }

        }

stage('Deploiement en dev'){
        environment
        {
        KUBECONFIG = credentials("config_Jenkins_exams") // we retrieve kubeconfig from secret file called config saved on jenkins
        }
            steps {

                script {
                sh '''
                  rm -Rf .kube
                  mkdir .kube
                  ls -alt   
                  cat $KUBECONFIG > .kube/config
                  cp fastapi/values1.yaml values1.yml
                  cp movieapi/values2.yaml values2.yaml
                  cat values1.yml
                  cat values2.yaml
                  sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values1.yml
                  sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values2.yaml
                  $HELM_PATH upgrade --install cast-service ./fastapi --values=values1.yml --namespace dev    
                  $HELM_PATH upgrade --install movie-service ./movieapi --values=values2.yaml --namespace dev
                  echo "STAGE SUCCESS : Deploiement sur l'environnement de développement effectué !"
                '''
                }
            }

        }

stage('Deploiement en QA'){
        environment
        {
        KUBECONFIG = credentials("config_Jenkins_exams") // we retrieve kubeconfig from secret file called config saved on jenkins
        }
            steps {

                script {
                sh '''
                  rm -Rf .kube
                  mkdir .kube
                  ls -alt
                  cat $KUBECONFIG > .kube/config
                  cp fastapi/values1.yaml values1.yml
                  cp movieapi/values2.yaml values2.yaml
                  cat values1.yml
                  cat values2.yaml
                  sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values1.yml
                  sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values2.yaml
                  $HELM_PATH upgrade --install cast-service ./fastapi --values=values1.yml --namespace qa
                  $HELM_PATH upgrade --install movie-service ./movieapi --values=values2.yaml --namespace qa
                  echo "STAGE SUCCESS : Deploiement sur l'environnement de test effectué !"
                '''
                }
            }

        }

stage('Deploiement en staging'){
        environment
        {
        KUBECONFIG = credentials("config_Jenkins_exams") // we retrieve  kubeconfig from secret file called config saved on jenkins
        }
            steps {

                script {
                sh '''
                  rm -Rf .kube
                  mkdir .kube
                  ls
                  cat $KUBECONFIG > .kube/config
                  cp fastapi/values1.yaml values1.yml
                  cp movieapi/values2.yaml values2.yaml
                  cat values1.yml
                  cat values2.yaml
                  sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values1.yml
                  sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values2.yaml
                  $HELM_PATH upgrade --install cast-service ./fastapi --values=values1.yml --namespace staging
                  $HELM_PATH upgrade --install movie-service ./movieapi --values=values2.yaml --namespace staging
                  echo "STAGE SUCCESS : Deploiement sur l'environnement de pré-poduction réussi !"
                '''
                }
            }

        }
  stage('Deploiement en prod'){
        environment
        {
        KUBECONFIG = credentials("config_Jenkins_exams") // we retrieve  kubeconfig from secret file called config saved on jenkins
        }
            steps {
            // Create an Approval Button with a timeout of 15minutes.
            // this require a manuel validation in order to deploy on production environment
                    timeout(time: 15, unit: "MINUTES") {
                        input message: 'Do you want to deploy in production ?', ok: 'Yes'
                    }

                script {
                sh '''
                  echo "USER APPROBATION SUCCESS : $DOCKER_ID give his/her authorization to deploy on prod environment"
                  rm -Rf .kube
                  mkdir .kube
                  ls
                  cat $KUBECONFIG > .kube/config
                  cp fastapi/values1.yaml values1.yml
                  cp movieapi/values2.yaml values2.yaml
                  cat values1.yml
                  cat values2.yaml
                  sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values1.yml
                  sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values2.yaml
                  $HELM_PATH upgrade --install cast-service ./fastapi --values=values1.yml --namespace prod
                  $HELM_PATH upgrade --install movie-service ./movieapi --values=values2.yaml --namespace prod
                  echo "STAGE SUCCESS : Deploiement sur l'environnement de production réussi !"
                  echo "END : Pipeline realized and triggered by $DOCKER_ID. Thank you for your courses, tips and advices! So Grateful !!"
                '''
                }
            }

        }
}
post {
    success {
        echo "This will run if the job succeeded"
        mail to: "hermann.acm@gmail.com",
             subject: "${env.JOB_NAME} - Build # ${env.BUILD_ID} is success",
             body: "Hi, this is Hermann, Jenkins pipeline was realized with success."
    }

    failure {
        echo "This will run if the job failed"
        mail to: "hermann.acm@gmail.com",
             subject: "${env.JOB_NAME} - Build # ${env.BUILD_ID} has failed",
             body: "Hi, this is Hermann, Jenkins pipeline failed. for more info on the pipeline failure, check out the console output at ${env.BUILD_URL}"
    }
}
}
