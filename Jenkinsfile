pipeline {
    agent any

    triggers {
        githubPush()
    }

    environment {
        DOCKERHUB_USER = 'slimane69'
        IMAGE_NAME = 'devops-mohamed-slimane'
        K8S_NAMESPACE = 'student-management'
        K8S_MANIFESTS_DIR = 'k8s'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/MedSlimane/devops-mohamed-slimane-4sim1'
            }
        }

        stage('Maven Compile') {
            steps {
                sh 'mvn compile'
            }
        }

        // stage('SonarQube Analysis') {
        //     steps {
        //         withSonarQubeEnv('xsonar') { // uses the global SonarQube server config
        //             sh 'mvn sonar:sonar'
        //         }
        //     }
        // }

        stage('Maven Package') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                    docker build -t ${DOCKERHUB_USER}/${IMAGE_NAME}:${BUILD_NUMBER} .
                    docker tag ${DOCKERHUB_USER}/${IMAGE_NAME}:${BUILD_NUMBER} ${DOCKERHUB_USER}/${IMAGE_NAME}:latest
                """
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds',
                                                 usernameVariable: 'USER',
                                                 passwordVariable: 'PASS')]) {
                    sh """
                        echo "$PASS" | docker login -u "$USER" --password-stdin
                    """
                                                 }
            }
        }

        stage('Docker Push') {
            steps {
                sh """
                    docker push ${DOCKERHUB_USER}/${IMAGE_NAME}:${BUILD_NUMBER}
                    docker push ${DOCKERHUB_USER}/${IMAGE_NAME}:latest
                """
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    sh """
                        kubectl apply -f ${K8S_MANIFESTS_DIR}/00-namespace.yaml
                        kubectl apply -n ${K8S_NAMESPACE} -f ${K8S_MANIFESTS_DIR}
                        kubectl -n ${K8S_NAMESPACE} set image deployment/student-management app=${DOCKERHUB_USER}/${IMAGE_NAME}:${BUILD_NUMBER}
                        kubectl -n ${K8S_NAMESPACE} rollout status deployment/student-management --timeout=180s
                    """
                }
            }
        }
    }
}
