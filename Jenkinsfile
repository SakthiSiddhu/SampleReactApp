pipeline {
    agent any
    tools {
        nodejs "NodeJS"
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/SakthiSiddhu/SampleReactApp'
            }
        }
        stage('Build') {
            steps {
                sh 'npm install --production'
            }
        }
        stage('Docker Build') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', passwordVariable: 'dockerHubPassword', usernameVariable: 'dockerHubUsername')]) {
                        sh """
                            docker build -t ratneshpuskar/samplereactapp:${env.BUILD_NUMBER} .
                            echo "${dockerHubPassword}" | docker login -u "${dockerHubUsername}" --password-stdin
                            docker push ratneshpuskar/samplereactapp:${env.BUILD_NUMBER}
                        """
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
                        name: samplereactapp
                    spec:
                        replicas: 1
                        selector:
                            matchLabels:
                                app: samplereactapp
                        template:
                            metadata:
                                labels:
                                    app: samplereactapp
                            spec:
                                containers:
                                - name: samplereactapp
                                  image: ratneshpuskar/samplereactapp:${env.BUILD_NUMBER}
                                  ports:
                                  - containerPort: 5000
                    """

                    def serviceYaml = """
                    apiVersion: v1
                    kind: Service
                    metadata:
                        name: samplereactapp
                    spec:
                        selector:
                            app: samplereactapp
                        ports:
                        - protocol: TCP
                          port: 5000
                          targetPort: 5000
                          nodePort: 30007
                    type: NodePort
                    """

                    sh """
                        ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@3.109.185.68 "kubectl apply -f -" <<< "${deploymentYaml}"
                        ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@3.109.185.68 "kubectl apply -f -" <<< "${serviceYaml}"
                    """
                }
            }
        }
    }
    post {
        success {
            echo 'Pipeline succeeded.'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}