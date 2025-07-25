pipeline {
    agent any

    environment {
        IMAGE_NAME   = "lendy-app"
        SECRET_KEY   = credentials('DJANGO_SECRET_KEY')  // Assuming you have a Django Secret Key stored as a Jenkins secret
         COMPOSE_FILE = "lendy/docker-compose.yml"
         DOCKER_HUB_REPO = 'marywam/lendy-app' // Docker Hub repository
     }

     stages {
        stage('Clone') {
             steps {
                echo "Cloning repository..."
                dir('lendy') {
                    git branch: 'main', url: 'https://github.com/mosesmbadi/lendy.git'
                 }
             }
         }

         stage('Install Dependencies & Test') {
             steps {
                 echo "Installing dependencies in the Jenkins container... python version matters"
                 // Uncomment and use the below if you need to install dependencies 
                 //dir('lendy') {
                    //sh '''
                        // pip install -r requirements.txt
                     // '''
                // }
             }
         }

         stage('Build Docker Image') {
             steps {
                 echo "Building Docker image..."
                 dir('lendy') {
                     sh '''
                         docker build -t $IMAGE_NAME .
                     '''
                 }
             }
        }

         stage('Push Docker Image to dockerhub') {
             steps {
                 echo "Pushing Docker image to Docker Hub..."
                 dir('lendy') {
                     script {
                         def dockerImageTag = "${DOCKER_HUB_REPO}:latest"
                         // Using credentials for Docker Hub login securely
                         withCredentials([usernamePassword(
                             credentialsId: 'DOCKER_HUB_REPO', 
                             usernameVariable: 'DOCKER_USER',  
                             passwordVariable: 'DOCKER_PASS' 
                         )]) {
                             sh """
                                 echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                                 docker tag $IMAGE_NAME $DOCKER_HUB_REPO:latest
                                 docker push $DOCKER_HUB_REPO:latest
                             """
                         }
                     }
                 }
             }
         }


         stage('Run Docker Image') {
             steps {
                 echo "run Docker image..."
                 sh '''
                     docker rm -f lendy_container || true
                     docker run -d --name lendy_container -p 8081:8080 $IMAGE_NAME
                 '''
             }
         }
     }

     post {
         success {
             echo "Build and deploy successful!"
         }
         failure {
             echo "Build or deploy failed."
         }
         always {
             echo "Pipeline finished."
         }
     }
 }

