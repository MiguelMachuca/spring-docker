pipeline {
  agent any
  environment {
    IMAGE_NAME = "demo-ci-cd:latest"
    VM_IP = "74.163.99.83"  // IP de tu VM
    VM_USER = "azureuser"   // Usuario de tu VM
    SSH_KEY = credentials('vm-ssh-key') // Credencial que debes configurar en Jenkins
    ARTIFACT_NAME = "target/demo-0.0.1-SNAPSHOT.jar"
    REMOTE_DIR = "/home/azureuser/artefactos/"
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
    stage('Deploy to VM') {
      steps {
        script {
          // Crear directorio remoto si no existe
          sh """
            ssh -o StrictHostKeyChecking=no -i ${SSH_KEY} ${VM_USER}@${VM_IP} \
            "mkdir -p ${REMOTE_DIR}"
          """
          
          // Copiar el artefacto
          sh """
            scp -o StrictHostKeyChecking=no -i ${SSH_KEY} \
            ${ARTIFACT_NAME} ${VM_USER}@${VM_IP}:${REMOTE_DIR}
          """
          
          // Detener aplicación anterior si existe y ejecutar la nueva
          sh """
            ssh -o StrictHostKeyChecking=no -i ${SSH_KEY} ${VM_USER}@${VM_IP} \
            "cd ${REMOTE_DIR} && \
            pkill -f 'java -jar' || true && \
            nohup java -jar \$(ls -t ${REMOTE_DIR}*.jar | head -1) > app.log 2>&1 &"
          """
        }
      }
    }
  }
  post {
    always {
      junit '**/target/surefire-reports/*.xml'
      archiveArtifacts artifacts: "${ARTIFACT_NAME}", fingerprint: true
    }
  }
}