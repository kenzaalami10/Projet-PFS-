pipeline {
    agent any

    environment {
        // Ajoutez d'autres variables d'environnement au besoin
        DOCKER_HUB_CREDENTIALS = credentials('dockerHubCredentials')
        MAVEN_TOOL_NAME = 'Maven'
    }

    tools {
        // Assurez-vous que l'outil Maven est configuré dans Jenkins avec le nom 'Maven'
        maven MAVEN_TOOL_NAME
    }

    stages {
        stage('Checkout SCM') {
            steps {
                checkout scm
            }
        }

        stage('Build Application') {
            steps {
                script {
                    echo '=== Building Petclinic Application ==='
                    // Vérifier si Maven est installé, sinon l'installer
                    sh "which mvn || { echo 'Maven not found, installing...'; sudo apt-get install -y maven; }"
                    sh "mvn -B -DskipTests clean package"
                }
            }
        }

        stage('Test Application') {
            steps {
                echo '=== Testing Petclinic Application ==='
                sh 'mvn test'
                post {
                    always {
                        junit 'target/surefire-reports/*.xml'
                    }
                }
            }
        }

        stage('Build Docker Image') {
            when {
                branch 'master'
            }
            steps {
                echo '=== Building Petclinic Docker Image ==='
                script {
                    app = docker.build("ibuchh/petclinic-spinnaker-jenkins")
                }
            }
        }

        // ... autres étapes du pipeline

        stage('Remove local images') {
            steps {
                echo '=== Delete the local docker images ==='
                sh("docker rmi -f ibuchh/petclinic-spinnaker-jenkins:latest || :")
                sh("docker rmi -f ibuchh/petclinic-spinnaker-jenkins:$SHORT_COMMIT || :")
            }
        }
    }

    post {
        success {
            echo 'Deployment successful!'
        }
        failure {
            echo 'Deployment failed.'
        }
    }
}

