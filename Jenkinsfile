pipeline {
    agent any
    stages {
        stage ('Build') {
            steps {
                script{
                     sh 'npm install'
                }
            }
        }
        stage ('SonarQube Analysis') {
            steps {
                script { 
                    def scannerHome = tool 'sonarscanner4'
                    withSonarQubeEnv('sonar-pro') {
                        sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=node.js-app"
                    }
                }
            }
        }
        stage('Docker Build') {
            steps {
                script {
                    sh 'docker build -t leehammesh/multi:v2 .'
                    sh 'docker images'
                }
            }
        }
        stage('Docker Push') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'dockerPass', variable: 'dockerPassword')]) {
                        sh "docker login -u leehammesh -p ${dockerPassword}"
                        sh 'docker push leehammesh/multi:v2'
                        sh 'docker rmi leehammesh/multi:v2'
                    }
                }
            }
        }
        stage('Deploy on k8s') {
            steps {
                script {
                    withKubeCredentials(kubectlCredentials: [[ credentialsId: 'kubernetes', namespace: 'ms' ]]) {
                        sh 'kubectl apply -f kube.yaml'
                        sh 'kubectl get pods -o wide'
                    }
                }
            }
        }
    }
}
