pipeline {
    agent any

    tools { 
        nodejs "NodeJS"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/SakthiSiddhu/SampleReactApp'
            }
        }
        stage('Build Project') {
            steps {
                sh 'npm install'
                sh 'npm run build --skip-tests'
            }
        }
        stage('Create Docker Image') {
            steps {
                script {
                    def repoName = "samplereactapp"
                    def buildTag = "ratneshpuskar/${repoName}:${env.BUILD_NUMBER}"
                    sh "docker build -t ${buildTag} ."
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', passwordVariable: 'DOCKER_HUB_PASSWORD', usernameVariable: 'DOCKER_HUB_USERNAME')]) {
                    sh 'echo $DOCKER_HUB_PASSWORD | docker login -u $DOCKER_HUB_USERNAME --password-stdin'
                    def repoName = 'samplereactapp'.toLowerCase()
                    def buildTag = "ratneshpuskar/${repoName}:${env.BUILD_NUMBER}"
                    sh "docker push ${buildTag}"
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    def repoName = 'samplereactapp'.toLowerCase()
                    def buildTag = "ratneshpuskar/${repoName}:${env.BUILD_NUMBER}"
                    writeFile file: 'deployment.yaml', text: """
                      apiVersion: apps/v1
                      kind: Deployment
                      metadata:
                        name: ${repoName}
                      spec:
                        replicas: 1
                        selector:
                          matchLabels:
                            app: ${repoName}
                        template:
                          metadata:
                            labels:
                              app: ${repoName}
                          spec:
                            containers:
                            - name: ${repoName}
                              image: ${buildTag}
                              ports:
                              - containerPort: 3000
                    """
                    writeFile file: 'service.yaml', text: """
                      apiVersion: v1
                      kind: Service
                      metadata:
                        name: ${repoName}
                      spec:
                        type: NodePort
                        ports:
                        - port: 3000
                          nodePort: 30007
                        selector:
                          app: ${repoName}
                    """
                    sh 'ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@65.1.86.218 "kubectl apply -f -" < deployment.yaml'
                    sh 'ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@65.1.86.218 "kubectl apply -f -" < service.yaml'
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment completed successfully!'
        }
        failure {
            echo 'Deployment failed.'
        }
    }
}