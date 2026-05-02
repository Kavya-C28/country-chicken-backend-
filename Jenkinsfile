pipeline {
    agent any
    tools {
        maven 'maven3.9.12'
        jdk 'java17'
    }
    environment {
        APP_NAME         = 'country-chicken-backend'
        NEXUS_MAVEN_URL  = '13.60.24.108:8081'
        NEXUS_DOCKER_URL = '13.60.24.108:8082'
        MAVEN_REPO       = 'maven-releases'
        DOCKER_REPO      = 'docker-releases'
        GROUP_ID         = 'com.countrychicken'
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'project-1',
                    url: 'https://github.com/Kavya-C28/country-chicken-backend-.git'
            }
        }
        stage('Set Version') {
            steps {
                script {
                    env.VERSION = sh(
                        script: "mvn help:evaluate -Dexpression=project.version -q -DforceStdout",
                        returnStdout: true
                    ).trim()
                    if (!env.VERSION) {
                        error "❌ Version not found from pom.xml"
                    }
                    env.JAR_NAME = "${APP_NAME}-${env.VERSION}.jar"
                    echo "✅ Version: ${env.VERSION}"
                }
            }
        }
        stage('Build JAR') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }
        stage('Upload JAR to Nexus') {
            steps {
                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: "${NEXUS_MAVEN_URL}",
                    groupId: "${GROUP_ID}",
                    version: "${env.VERSION}",
                    repository: "${MAVEN_REPO}",
                    credentialsId: 'nexus-credentials',
                    artifacts: [
                        [
                            artifactId: "${APP_NAME}",
                            classifier: '',
                            file: "target/${env.JAR_NAME}",
                            type: 'jar'
                        ]
                    ]
                )
            }
        }
        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${NEXUS_DOCKER_URL}/${DOCKER_REPO}/${APP_NAME}:${env.VERSION} ."
            }
        }
        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-nexus-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                        echo "\$DOCKER_PASS" | docker login ${NEXUS_DOCKER_URL} -u "\$DOCKER_USER" --password-stdin
                        docker push ${NEXUS_DOCKER_URL}/${DOCKER_REPO}/${APP_NAME}:${env.VERSION}
                        docker logout ${NEXUS_DOCKER_URL}
                    """
                }
            }
        }
    }
    post {
        success {
            echo "✅ Build & Push Successful"
        }
        failure {
            echo "❌ Build Failed"
        }
        always {
            sh 'docker system prune -f'
            cleanWs()
        }
    }
}
