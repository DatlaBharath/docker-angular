pipeline {
    agent any
    tools {
        nodejs 'NodeJS'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/DatlaBharath/docker-angular'
            }
        }

        stage('Build') {
            steps {
                sh 'npm install -g @angular/cli'
                sh 'npm install'
                sh 'npx ng build --prod'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    def imageName = "sakthisiddu1/docker-angular:${env.BUILD_NUMBER}"
                    sh "docker build -t ${imageName} ."
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                        sh 'echo ${DOCKER_PASSWORD} | docker login -u ${DOCKER_USERNAME} --password-stdin'
                         def imageName = "sakthisiddu1/docker-angular:${env.BUILD_NUMBER}"
                        sh "docker push ${imageName}"
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    def deploymentYaml = """
apiVersion: apps/v1
kind: Deployment
metadata:
  name: docker-angular-deployment
  labels:
    app: docker-angular
spec:
  replicas: 1
  selector:
    matchLabels:
      app: docker-angular
  template:
    metadata:
      labels:
        app: docker-angular
    spec:
      containers:
      - name: docker-angular
        image: sakthisiddu1/docker-angular:${env.BUILD_NUMBER}
        ports:
        - containerPort: 80
"""

                    def serviceYaml = """
apiVersion: v1
kind: Service
metadata:
  name: docker-angular-service
spec:
  selector:
    app: docker-angular
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
    nodePort: 30007
  type: NodePort
"""

                    sh """echo "$deploymentYaml" > deployment.yaml"""
                    sh """echo "$serviceYaml" > service.yaml"""

                    sh 'ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@52.66.80.80 "kubectl apply -f -" < deployment.yaml'
                    sh 'ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@52.66.80.80 "kubectl apply -f -" < service.yaml'
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment was successful'
        }
        failure {
            echo 'Deployment failed'
        }
    }
}