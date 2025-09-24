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
        stage("Deploy"){
            steps{
                echo "Deploying the Application"
                sh "docker run -d -p 3000:3000 ${IMAGE_NAME}:${params.IMAGE_TAG}"
                }
            }
        
    }
}