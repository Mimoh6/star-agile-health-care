pipeline {
    agent any

    stages {
        stage('Git Checkout') {
            steps {
                echo 'This stage is to clone the repo from github'
                git branch: 'master', url: 'https://github.com/Mimoh6/star-agile-health-care.git'
            }
        }

        stage('Create Package') {
            steps {
                echo 'This stage will compile, test, package my application'
                sh 'mvn package'
            }
        }

        stage('Create Docker Image') {
            steps {
                echo 'This stage will Create a Docker image'
                sh 'docker build -t mimoh6/health-care:1.0 .'
            }
        }

        stage('Docker-Login') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-25', passwordVariable: 'dockerpasswd', usernameVariable: 'dockerusername')]) {
                    sh '''
                        echo "$dockerpasswd" | docker login -u "$dockerusername" --password-stdin
                    '''
                }
            }
        }

        stage('Docker Push-Image') {
            steps {
                echo 'This stage will push my new image to the dockerhub'
                sh 'docker push mimoh6/health-care:1.0'
            }
        }

        stage('AWS-Login') {
            steps {
                withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'awslogin', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    echo 'Authenticating with AWS'
                    sh 'aws sts get-caller-identity'
                }
            }
        }

        stage('setting the Kubernetes Cluster') {
            steps {
                withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'awslogin', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    dir('terraform_files') {
                        sh 'terraform init'
                        sh 'terraform validate'
                        sh 'terraform apply --auto-approve'
                        sh 'sleep 20'
                    }
                }
            }
        }

        stage('deploy kubernetes') {
            steps {
                sh 'sudo chmod 600 ./terraform_files/LinuxUbuntu.pem'
                sh 'minikube start'
                sh 'sleep 30'
                sh 'sudo scp -o StrictHostKeyChecking=no -i ./terraform_files/LinuxUbuntu.pem deployment.yml ubuntu@172.31.23.75:/home/ubuntu/'
                sh 'sudo scp -o StrictHostKeyChecking=no -i ./terraform_files/LinuxUbuntu.pem service.yml ubuntu@172.31.23.75:/home/ubuntu/'
                script {
                    try {
                        sh 'ssh -o StrictHostKeyChecking=no -i ./terraform_files/LinuxUbuntu.pem ubuntu@172.31.23.75 kubectl apply -f .'
                    } catch (error) {
                        sh 'ssh -o StrictHostKeyChecking=no -i ./terraform_files/LinuxUbuntu.pem ubuntu@172.31.23.75 kubectl apply -f .'
                    }
                }
            }
        }
    }
}
