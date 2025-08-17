pipeline {
  agent any
  options { timestamps(); ansiColor('xterm') }

  stages {
    stage('Checkout') {
      steps {
        echo "=== 检出代码 ==="
        checkout scm
      }
    }

    stage('Setup venv & Install deps') {
      steps {
        echo "=== 安装依赖 ==="
        sh '''#!/usr/bin/env bash
        set -euxo pipefail

        # 统一用绝对路径，避免激活失效
        if [ ! -d .venv ]; then
          python3 -m venv .venv
        fi

        # 升级 pip & 安装依赖
        ./.venv/bin/python -m pip install --upgrade pip
        ./.venv/bin/pip install -r requirements.txt
        '''
      }
    }

    stage('Migrate') {
      steps {
        echo "=== 执行 migrate ==="
        sh '''#!/usr/bin/env bash
        set -euxo pipefail
        ./.venv/bin/python manage.py migrate --noinput
        '''
      }
    }

    stage('Run Django Server') {
      steps {
        echo "=== 启动 runserver ==="
        sh '''#!/usr/bin/env bash
        set -euxo pipefail

        # 目录固定到当前 workspace（Jenkins 默认就是这里）
        pwd

        # 尝试通过 PID 文件精确杀；若无再兜底
        if [ -f runserver.pid ]; then
          PID=$(cat runserver.pid || true)
          if [ -n "${PID:-}" ] && kill -0 "$PID" 2>/dev/null; then
            kill "$PID" || true
            # 给 3 秒优雅退出
            for i in $(seq 1 6); do
              kill -0 "$PID" 2>/dev/null || break
              sleep 0.5
            done
            kill -9 "$PID" 2>/dev/null || true
          fi
        fi

        # 兜底用端口把关（可能需要安装 psmisc：apt-get install -y psmisc）
        fuser -k 443/tcp 2>/dev/null || true

        # 启动（禁用重载，写 PID & 日志）
        nohup ./.venv/bin/python manage.py runserver 0.0.0.0:443 --noreload --verbosity 2 > runserver.log 2>&1 &
        echo $! > runserver.pid

        # 健康检查（最多等 15 秒）
        for i in $(seq 1 30); do
          if curl -sI --max-time 1 http://127.0.0.1:443/ >/dev/null; then
            echo "Service is UP."
            break
          fi
          sleep 0.5
        done

        # 若仍然不通，则打印日志并失败
        if ! curl -sI --max-time 1 http://127.0.0.1:443/ >/dev/null; then
          echo "Service failed to start. Last 200 lines of runserver.log:"
          tail -n 200 runserver.log || true
          exit 1
        fi

        echo "=== runserver.log 尾部 ==="
        tail -n 80 runserver.log || true
        '''
      }
    }
  }

  // 可选：只在 main 分支跑部署
  // triggers { pollSCM('* * * * *') } // 或 GitHub webhook
  // post { always { archiveArtifacts artifacts: 'runserver.log', onlyIfSuccessful: false } }
}
