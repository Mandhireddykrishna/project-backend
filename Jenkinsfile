pipeline {
    agent {
        label 'jumpserver'
    }

    environment {
        KUBECONFIG_CREDENTIAL_ID = 'k8s-config'
        version = "backend_${env.BUILD_NUMBER}"
        docker_image = "krishn1/moviestreaming:${version}"

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'git@github.com:Mandhireddykrishna/project-backend.git'
            }
        }

       stage('Login to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh "echo \"$DOCKER_PASSWORD\" | sudo docker login --username \"$DOCKER_USERNAME\" --password-stdin"
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    def dockerfilePath = '.'
                   sh "sudo docker build -t 'krishn1/moviestreaming:${version}' ."
                }
            }
        }
        
        stage('Push Docker Image') {
            steps {
                script {
                    sh "sudo docker push 'krishn1/moviestreaming:${version}'"
                }
            }
        } 
        stage('Security Scan') {
            steps {
                script {
                    def outputFilePath = "${env.WORKSPACE}/trivy_scan.txt"
                    sh "sudo trivy image ${docker_image} > ${outputFilePath}"
                    sh "cat ${outputFilePath}"
                }
            }
        }
        stage('Cleanup Docker Images') {
            steps {
                script {
                    sh "sudo docker rmi -f ${env.docker_image}"
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {

                withCredentials([file(credentialsId: KUBECONFIG_CREDENTIAL_ID, variable: 'KUBECONFIG')]) {
                    script {
                        def kubeconfigPath = env.KUBECONFIG
                      withEnv(["VERSION=${env.version}"])
                      {
                         sh "echo ${VERSION}"
                         sh "export KUBECONFIG=${kubeconfigPath}"
                         //sh "kubectl scale deploy api --replicas=0 -n three-tier"
                         sh" sed -i 's/VERSION/${VERSION}/g' backend.yml"
                         sh " cat backend.yml"
                         sh "kubectl apply -f backend.yml --validate=false"
                         sh "kubectl get pods -n three-tier"
                         sh "kubectl get svc -n three-tier"
                      }
                    }
                }
            }
        }
    }
}
