pipeline {
    agent any

    stages {
        stage('Git Checkout') {
            steps {
                echo 'This stage is to clone the repo from github'
                git branch: 'master', url: 'https://github.com/Pratap-mulakala/star-agile-health-care.git'
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
                dir('terraform_files') {
                    sh 'terraform init'
                    sh 'terraform validate'
                    sh 'terraform apply --auto-approve'
                    sh 'sleep 20'
                }
            }
        }
    }
}
