pipeline {
  agent any
  triggers { githubPush() }
  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }
    stage('Run Django Server') {
      steps {
        sh '''
          # 杀掉旧的 runserver（如果有）
          pkill -f "manage.py runserver -8888" || true

          # 启动新的 runserver 到后台
          nohup python3 manage.py runserver 0.0.0.0:8888 > runserver.log 2>&1 &
        '''
      }
    }
  }
}
