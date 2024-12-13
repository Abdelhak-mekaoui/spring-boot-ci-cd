pipeline {
    agent any

    tools {
        maven 'Maven 3.9.8'
    }

    environment {
        BUILD_VERSION = readMavenPom(file: 'pom.xml').getVersion()
        DOCKER_IMAGE = 'mekaouiabdelhak1/spring-boot-ci-cd'
    }

    stages {

        stage('Clean Workspace') {
            steps {
                script {
                    cleanWs()
                }
            }
        }

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Abdelhak-mekaoui/spring-boot-ci-cd'
            }
        }

        stage('Unit Testing') {
            steps {
                sh 'mvn clean test'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv(credentialsId: 'sonarQube-token', installationName: 'SonarQube') {
                    sh "mvn verify sonar:sonar -DskipTests=true -Dsonar.projectKey=spring-boot-ci-cd -Dsonar.projectName='spring-boot-ci-cd'"
                }

            }
        }

        stage('Package') {
            steps {
                sh 'mvn package -Dmaven.test.skip'
            }
        }

        stage('Publish test results') {
            steps {
                junit "**/target/surefire-reports/*.xml"
            }
        }

        stage('Trivy FS Scan') {
            steps {
                sh 'trivy fs .'
            }
        }

        stage('Docker Build') {
            steps {
                sh "mvn spring-boot:build-image -DskipTests"
            }
        }

        stage('Trivy Image Scan') {
            steps {
                sh 'trivy image --scanners vuln mekaouiabdelhak1/spring-boot-ci-cd:${BUILD_VERSION}'
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'dockerHubUsername', passwordVariable: 'dockerHubPassword')]) {
                    sh "docker login -u ${env.dockerHubUsername} -p ${env.dockerHubPassword}"
                    sh "docker push mekaouiabdelhak1/spring-boot-ci-cd:${BUILD_VERSION}"
                }
            }
        }

        stage('Trigger microservices-k8s-manifests Job') {
            steps {
                sh "curl -v -k --user admin:11295d15cb5acb2914d803b4d62222b728 -X POST -H 'cache-control: no-cache' -H 'Content-Type: application/x-www-form-urlencoded' --data 'DOCKER_IMAGE=${DOCKER_IMAGE}&BUILD_VERSION=${BUILD_VERSION}&APPLICATION=spring-boot-testing' http://localhost:8080/job/microservices-k8s-manifests/buildWithParameters?token=spring-microservices-in-action-token"
            }
        }
    }
}
