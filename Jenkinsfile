pipeline {
  agent any
  options { timestamps(); ansiColor('xterm') }
  triggers { githubPush() }   // ← 原生 GitHub 触发
  stages {
    stage('Checkout') {
      steps {
        checkout scm // 任务从SCM创建则可用；手写则用 GitSCM 同上
      }
    }
    stage('Build') {
      steps { sh 'echo build here' }
    }
  }
}
