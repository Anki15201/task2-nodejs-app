pipeline {
    agent any

    environment {
        IMAGE_NAME = 'anki15201/nodejs-app'
    }

    parameters {
        string(name: 'IMAGE_TAG', defaultValue: '', description: 'setting docker image tags')
    }

    tools {
        nodejs 'NodeJS' // Name of your NodeJS installation in Jenkins
    }

     stages{
        stage("Validate Parameters"){
            steps{
                script {
                    if (params.IMAGE_TAG == '') {
                        error ("IMAGE_TAG must be provided")
                    }
                }
            }
        }
        stage("Checkout"){
            steps{
                checkout scm
            }
        }
        stage("Test") {
            steps {
                script {
                    echo "Running tests on Jenkins agent..."
                    sh '''
                        set -e
                        npm install
                        npm test
                    '''
                    echo "Test executed successfully"
                }
            }
        }
        stage("Build : Image"){
            steps{
                script {
                    echo "Building docker images"
                    sh "docker build -t ${IMAGE_NAME}:${params.IMAGE_TAG} ."
                    echo "Image build is successfull"
                }
            }
        }
        stage("Push to DockerHub"){
            steps{
                echo "Pushing image to dockerhub"
                withCredentials([usernamePassword(credentialsId: 'dockerhubcred', 
                                                  usernameVariable: 'DOCKER_USER', 
                                                  passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    sh "docker push ${IMAGE_NAME}:${params.IMAGE_TAG}"
                }
            }
        }
        stage("Deploy") {
    steps {
        script {
            echo "Deploying the Application..."

            // Stop old container if exists
            sh '''
                if [ "$(docker ps -aq -f name=nodejs-app)" ]; then
                  docker stop nodejs-app || true
                  docker rm nodejs-app || true
                fi
            '''

            // Run new container
            sh "docker run -d --name nodejs-app -p 3000:3000 ${IMAGE_NAME}:${params.IMAGE_TAG}"
            echo "Application deployed successfully at http://<jenkins-server-ip>:3000"
        }
    }
}
        
    }
}