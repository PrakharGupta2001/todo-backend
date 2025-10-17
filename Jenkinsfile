pipeline {
  agent any

  environment {
    SSH_CRED    = 'deploy-ssh-key'    // Jenkins credential id (SSH Username with private key)
    DEPLOY_USER = 'deploy'
    REMOTE_DIR  = '/opt/todo-backend'
    DEPLOY_HOST = '54.234.241.41'     // backend server IP (master deploy target)
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Build') {
      steps {
        sh 'mvn -B clean package -DskipTests'
      }
    }

    stage('Archive') {
      steps {
        archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
      }
    }

    stage('Deploy') {
      steps {
        sshagent (credentials: ['deploy-ssh-key']) {
          sh '''
            set -e
            scp -o StrictHostKeyChecking=no target/*.jar ${DEPLOY_USER}@${DEPLOY_HOST}:${REMOTE_DIR}/app.jar
            ssh -o StrictHostKeyChecking=no ${DEPLOY_USER}@${DEPLOY_HOST} 'bash ${REMOTE_DIR}/deploy.sh'
          '''
        }
      }
    }
  }

  post {
    success { echo "Backend deployed to ${env.DEPLOY_HOST}" }
    failure { echo "Deployment failed" }
  }
}
