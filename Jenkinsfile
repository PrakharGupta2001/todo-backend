pipeline {
  agent any
  environment {
    SSH_CRED = 'deploy-ssh-key'
    DEPLOY_USER = 'deploy'
    REMOTE_DIR = '/opt/todo-backend'
    BACKEND_DEV = '54.234.241.41'    // you can reuse same for all branches if you want
    BACKEND_STAGE = '54.234.241.41'
    BACKEND_PROD = '54.234.241.41'
  }
  stages {
    stage('Checkout') { steps { checkout scm } }
    stage('Build') { steps { sh 'mvn -B clean package -DskipTests' } }
    stage('Archive') { steps { archiveArtifacts artifacts: 'target/*.jar', fingerprint: true } }
    stage('Select Host') {
      steps { script {
        if (!env.BRANCH_NAME) { env.DEPLOY_HOST = env.BACKEND_PROD } 
        else if (env.BRANCH_NAME == 'main') { env.DEPLOY_HOST = env.BACKEND_PROD } 
        else if (env.BRANCH_NAME == 'staging') { env.DEPLOY_HOST = env.BACKEND_STAGE } 
        else { env.DEPLOY_HOST = env.BACKEND_DEV }
        echo "Deploying branch ${env.BRANCH_NAME} to ${env.DEPLOY_HOST}"
      }}
    }
    stage('Deploy') {
      steps {
        sshagent (credentials: [env.SSH_CRED]) {
          sh """
            scp target/*.jar ${DEPLOY_USER}@${DEPLOY_HOST}:${REMOTE_DIR}/app.jar
            ssh ${DEPLOY_USER}@${DEPLOY_HOST} 'bash ${REMOTE_DIR}/deploy.sh'
          """
        }
      }
    }
  }
  post {
    success { echo "Backend deployed to ${env.DEPLOY_HOST}" }
    failure { echo "Deployment failed" }
  }
}
