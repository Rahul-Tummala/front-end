pipeline {
    agent any

    // tools {
    //     maven 'Maven3'
    // }

    environment {
        AWS_ACCOUNT_ID = "891377337960"
        AWS_DEFAULT_REGION = "us-east-1"
        IMAGE_REPO_NAME = "frontend"
        IMAGE_TAG = "latest"
        REPOSITORY_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}"
        GIT_REPO_URL = "https://github.com/navitascomet2/front-end/"
        // SONAR_PROJECT_KEY = "Rahul-Tummala_Sample-App-Sonarqube_5f973a9a-b3f9-423d-9d78-b888a2131481"
        // SONAR_PROJECT_NAME = "Sample-App-Sonarqube"
        K8S_CREDENTIALS_ID = "K8S-3"
    }

    stages {
        stage('Cloning Git') {
            steps {
                checkout([
                    $class: 'GitSCM', 
                    branches: [[name: '*/main']], 
                    doGenerateSubmoduleConfigurations: false, 
                    extensions: [], submoduleCfg: [],
                    userRemoteConfigs: [
                        [
                            credentialsId: 'github', 
                            url: "${GIT_REPO_URL}"
                        ]
                    ]
                ])     
            }
        }

        // stage('SonarQube Analysis') {
        //     steps {
        //         script {
        //             def mvn = tool 'Maven3'
        //             withSonarQubeEnv() {
        //                 sh "${mvn}/bin/mvn clean verify sonar:sonar -Dsonar.projectKey=${SONAR_PROJECT_KEY} -Dsonar.projectName='${SONAR_PROJECT_NAME}'"
        //             }
        //         }
        //     }
        // }

        //  stage('Build') {
        //     steps {
        //         nodejs(nodeJSInstallationName: 'nodeJS_22.0.0') {
        //             sh 'npm install'
        //         }
        //     }
        // }

        stage('Building image') {
            steps {
                script {
                    dockerImage = docker.build "${REPOSITORY_URI}:${IMAGE_TAG}"
                }
            }
        }

        stage('Pushing to ECR') {
            steps {  
                script {
                    sh "aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${REPOSITORY_URI}"
                    sh "docker tag ${REPOSITORY_URI}:${IMAGE_TAG} ${REPOSITORY_URI}:${BUILD_NUMBER}"
                    sh "docker push ${REPOSITORY_URI}:${IMAGE_TAG}"
                    sh "docker push ${REPOSITORY_URI}:${BUILD_NUMBER}"
                }
            }
        }

        stage('K8S Deploy') {
            steps {   
                script {
                    withKubeConfig([credentialsId: "${K8S_CREDENTIALS_ID}", serverUrl: '']) {
                        // sh 'kubectl apply -f eks-deploy-k8s.yaml'
                        // sh 'kubectl apply -f pv.yaml'
                        // sh 'kubectl apply -f pvc.yaml'
                        // sh 'kubectl apply -f deployment.yaml'
                        // sh 'kubectl apply -f service.yaml'
                        sh 'kubectl apply -f combined.yaml'
                        sh 'kubectl rollout restart deployment navitas-frontend'
                    }
                }
            }
        }
    }
}
