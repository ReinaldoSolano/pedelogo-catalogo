pipeline {
    agent any

    stages {

        stage('Get Source') {
            steps {
                git url: 'https://github.com/ReinaldoSolano/pedelogo-catalogo.git', branch: 'main'
            }
        }

        stage('Docker Build Image') {
            steps {
                script {
                    dockerapp = docker.build("rssystem/pedelogo-catalogo:${env.BUILD_ID}",
                      '-f ./src/PedeLogo.Catalogo.Api/Dockerfile .')
                }
            }
        }

        stage('Docker Push Image') {
            steps {
                script {
                        docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
                        dockerapp.push('latest')
                        dockerapp.push("${env.BUILD_ID}")
                    }
                }
            }
        }

        stage('Deploy Kubernetes- k8s') {
            agent {
                kubernetes {
                    cloud 'kubernetes'
                }
            }
            environment {
                tag_version = "${env.BUILD_ID}"
            }

            steps {
                sh 'sed -i "s/{{tag}}/$tag_version/g" ./k8s/api.yaml'
                sh 'cat ./k8s/api.yaml'
                kubernetesDeploy(configs: '**/k8s/**', kubeconfigId: 'kubeconfig')
            }
        }
    }
}