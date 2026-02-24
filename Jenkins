pipeline {
  agent any

  environment {
    REPO_DIR        = "${WORKSPACE}"
    FRONTEND_DIR    = "${WORKSPACE}/frontend"
    BACKEND_DIR     = "${WORKSPACE}/backend"

    WEB_ROOT        = "/var/www/food-frontend"
    BACKEND_PORT    = "5000"
    PM2_NAME        = "food-backend"
  }

  stages {

    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Frontend: Install') {
      steps {
        dir("${FRONTEND_DIR}") {
          sh 'npm ci || npm install'
        }
      }
    }

    stage('Frontend: Build') {
      steps {
        dir("${FRONTEND_DIR}") {
          sh 'npm run build'
          sh 'test -d dist'
        }
      }
    }

    stage('Frontend: Deploy') {
      steps {
        sh "sudo rsync -a --delete ${FRONTEND_DIR}/dist/ ${WEB_ROOT}/"
      }
    }

    stage('Backend: Install') {
      steps {
        dir("${BACKEND_DIR}") {
          sh 'npm ci || npm install'
        }
      }
    }

    stage('Backend: Start/Restart (PM2)') {
      steps {
        dir("${BACKEND_DIR}") {
          // Assumes backend has "start" script in package.json
          sh """
            pm2 describe ${PM2_NAME} >/dev/null 2>&1 && pm2 delete ${PM2_NAME} || true
            PORT=${BACKEND_PORT} pm2 start npm --name ${PM2_NAME} -- start
            pm2 save
          """
        }
      }
    }

    stage('Nginx: Reload') {
      steps {
        sh 'sudo nginx -t'
        sh 'sudo systemctl reload nginx'
      }
    }

    stage('Smoke Test') {
      steps {
        sh "curl -I http://127.0.0.1 | head -n 5"
        sh "curl -i http://127.0.0.1/api/ | head -n 20 || true"
      }
    }
  }
}
