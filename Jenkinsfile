pipeline {
    agent any

    environment {
        DOCKERHUB_USER = 'amarnath2413'
        IMAGE_NAME = "${DOCKERHUB_USER}/${repo_name}"
        //PRIVATE_KEY = 'private_key'// Jenkins Credential ID (SSH private key)
        EC2_HOST = 'ec2-user@16.170.254.164'
        REMOTE_APP_PATH = '/home/ec2-user/img-building'
        service_name=""
    }

    stages {
        
        stage('Clone Repository') {
            
            steps {
                git url:'https://github.com/Amarnath-G/${repo_name}.git',branch:'main',credentialsId:'github-cred'
                
            //     script{
            //     if(repo_name=="mern-app_server"){
            //         env.service_name="server"
            //     }
            //     else{
            //         env.service_name="client"
            //     }
            // }
            }
            
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME .'
            }
        }

        stage('Tag & Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin &&
                    docker tag $IMAGE_NAME ${IMAGE_NAME}:${env.BUILD_NUMBER} &&
                    docker push $IMAGE_NAME
                    """
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                withCredentials([file(credentialsId: 'private_key', variable: 'SSH_KEY')]) {
                    sh """
                    echo "scp entering"

                    scp -i $SSH_KEY docker-compose.yml $EC2_HOST:$REMOTE_APP_PATH

                    echo "scp completed"
                    ssh -o StrictHostKeyChecking=no -i $SSH_KEY $EC2_HOST '
                        cd $REMOTE_APP_PATH &&
                        docker compose pull $repo_name &&
                        docker compose up -d $repo_name &&
                        echo "Running $repo_name" &&
                        docker ps
                    '
                    """
                }
            }
        }
    }

    post {
        failure {
            echo "Deployment failed."
        }
        success {
            echo "Deployment succeeded."
        }
        always{
            cleanWs() 
        }
    }
}






// pipeline {
//     agent any

//     environment {
//         DOCKER_HOST_USER = 'ec2-user'                         // or ubuntu, root, etc.
//         DOCKER_HOST_IP = '16.170.254.164'
//         PRIVATE_KEY = 'private_key'                       // Jenkins Credentials ID (file)
//         REPO_PATH = '/home/ec2-user/app1'                    // Path to code on Docker host
//     }

//     stages {
//         stage('Checkout') {
//             steps {
//                 checkout scm
//             }
//         }

//         stage('Detect Changes') {
//             steps {
//                 script {
//                     def changeLog = currentBuild.changeSets
//                     def frontendChanged = false
//                     def backendChanged = false

//                     echo "$changeLog.size()"
//                     if (!changeLog || changeLog.size() == 0) {
//                         // First run or no changes detected
//                         frontendChanged = true
//                         backendChanged = true
//                         echo "No changelog detected. Forcing full build."
//                     } 
//                     else{
//                     for (changeSet in changeLog) {
//                         for (entry in changeSet.items) {
//                             for (file in entry.affectedFiles) {
//                                 if (file.path.startsWith("client/")) {
//                                     frontendChanged = true
//                                 } else if (file.path.startsWith("server/")) {
//                                     backendChanged = true
//                                 }
//                             }
//                         }
//                     }
//                     }

//                     env.FRONTEND_CHANGED = frontendChanged.toString()
//                     env.BACKEND_CHANGED = backendChanged.toString()
//                 }
//             }
//         }

//         stage('Build & Restart on Docker Host') {
//             steps {
//                 withCredentials([file(credentialsId: env.PRIVATE_KEY, variable: 'KEY_PATH')]) {
//                     script {
//                         if (env.FRONTEND_CHANGED == 'true') {
//                             sh """
//                                 ssh -o StrictHostKeyChecking=no -i $KEY_PATH $DOCKER_HOST_USER@$DOCKER_HOST_IP '
//                                 cd $REPO_PATH &&
//                                 git pull origin main &&
//                                 docker-compose build client &&
//                                 docker-compose up -d --no-deps --build client'
//                             """
//                         }

//                         if (env.BACKEND_CHANGED == 'true') {
//                             sh """
//                                 ssh -o StrictHostKeyChecking=no -i $KEY_PATH $DOCKER_HOST_USER@$DOCKER_HOST_IP '
//                                 cd $REPO_PATH &&
//                                 git pull origin main &&
//                                 docker-compose build server &&
//                                 docker-compose up -d --no-deps --build server'
//                             """
//                         }
//                     }
//                 }
//             }
//         }
//     }
// }
