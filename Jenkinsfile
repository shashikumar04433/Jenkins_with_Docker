pipeline {
    environment {
        imagename = "shashikumar023/jenkinss"
        dockerImage = ''
        containerName = 'my-container'
        secretName = 'Docker_Login_Details'
        dockerImageTag = "${imagename}:${env.BUILD_NUMBER}"
        AWS_REGION = 'ap-south-1'  // Adjust this based on your AWS region
        awsCredentialsId = 'awscredentials'  // Your AWS credentials ID
    }

    agent any

    stages {
        stage('Cloning Git') {
            steps {
                git(url: 'https://github.com/shashikumar04433/Jenkins_with_Docker.git', branch: 'main')
            }
        }

        stage('Building image') {
            steps {
                script {
                    dockerImageTag = "${imagename}:${env.BUILD_NUMBER}"
                    dockerImage = docker.build dockerImageTag
                }
            }
        }

        stage('Running image') {
            steps {
                script {
                    sh "docker run -d --name ${containerName} ${dockerImageTag}"
                }
            }
        }

        stage('Stop and Remove Container') {
            steps {
                script {
                    sh "docker stop ${containerName} || true"
                    sh "docker rm ${containerName} || true"
                }
            }
        }

        stage('Fetch Docker Hub Credentials') {
            steps {
                script {
                    withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: awsCredentialsId]]) {
                        def secretJson = sh(script: "aws secretsmanager get-secret-value --region ${AWS_REGION} --secret-id ${secretName} --query SecretString --output text", returnStdout: true).trim()
                        def secret = readJSON text: secretJson
                        env.DOCKER_USERNAME = secret.username
                        env.DOCKER_PASSWORD = secret.password
                    }
                }
            }
        }

        stage('Deploy Image') {
            steps {
                script {
                    sh "docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD"
                    sh "docker push ${dockerImageTag}"
                }
            }
        }
    }
}
