pipeline {
  agent any
  environment {
    IMAGE_NAME = "demo-ci-cd:latest"
    STAGING_SERVER = "ssh_server"
    ARTIFACT_NAME = "target/demo-0.0.1-SNAPSHOT.jar"
    DEPLOY_USER = "root"
    REMOTE_DIR = "/home/ubuntu/artefactos/spring-docker/"
  }
  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }
    stage('Build & Test') {
      steps {
        sh 'mvn -B clean package'
      }
    }
    stage('Build Docker Image') {
      steps {
        sh 'docker build -t $IMAGE_NAME .'
      }
    }
    stage('Run Container') {
      steps {
        sh 'docker rm -f demo-ci-cd || true'
        sh 'docker run -d --name demo-ci-cd -p 8080:8080 $IMAGE_NAME'
      }
    }
  }
  post {
    always {
      junit '**/target/surefire-reports/*.xml'
      archiveArtifacts artifacts: "${ARTIFACT_NAME}", fingerprint: true
    }
    success {
      sh 'ls -la target/'
      sh "scp ${ARTIFACT_NAME} ${DEPLOY_USER}@${STAGING_SERVER}:${REMOTE_DIR}"
      sh "ssh ${DEPLOY_USER}@${STAGING_SERVER} 'cd ${REMOTE_DIR} && nohup java -jar \$(ls -t *.jar | head -1) > app.log 2>&1 &'"
    }
  }
}