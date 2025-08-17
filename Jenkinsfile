pipeline {
  agent any
  stages {
    stage('Checkout') {
      steps {
        echo "=== 开始检出代码 ==="
        checkout scm
        echo "=== 检出完成 ==="
      }
    }

    stage('Setup venv & Install deps') {
      steps {
        echo "=== 开始安装依赖 ==="
        sh '''
          set -eux
          if [ ! -d .venv ]; then
            python3 -m venv .venv
          fi
          . .venv/bin/activate
          python -m pip install --upgrade pip
          pip install -r requirements.txt
        '''
        echo "=== 依赖安装完成 ==="
      }
    }

    stage('Migrate & Collectstatic') {
      steps {
        echo "=== 开始执行 migrate 和 collectstatic ==="
        sh '''
          set -eux
          . .venv/bin/activate
          python manage.py migrate --noinput
          python manage.py collectstatic --noinput
        '''
        echo "=== 数据库迁移和静态文件收集完成 ==="
      }
    }

    stage('Run Django Server') {
      steps {
        echo "=== 开始启动 Django runserver ==="
        sh '''
          set -eux
          pkill -f "manage.py runserver 0.0.0.0:8888" || true
          . .venv/bin/activate
          nohup python manage.py runserver 0.0.0.0:8888 > runserver.log 2>&1 &
          sleep 3
          echo "=== runserver.log 内容 ==="
          tail -n 50 runserver.log || true
        '''
        echo "=== runserver 启动命令已执行 ==="
      }
    }
  }
}
