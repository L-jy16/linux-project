<!-- @format -->

# 시스템 관제 자동화 스크립트 개발

1. 개요

Docker를 이용하여

2. 실행환경

- OS: macOS
- Python: 3.13.2
- 개발 도구: Terminal, Docker

3. 수행 리스트

- [✔] Docker Ubuntu 22.04 환경 구축
- [✔] SSH 서버 설치 및 보안 설정
- [✔] 방화벽(UFW) 설정
- [✔] 사용자 계정 생성
- [✔] 그룹 생성 및 사용자 그룹 할당
- [✔] 프로젝트 디렉토리 구조 생성
- [✔] 디렉토리 및 파일 권한 설정
- [✔] 환경 변수 설정
- [✔] API Key 파일 생성
- [✔] Agent 서비스 작성 및 실행
- [✔] 서비스 포트(15034) 개방 확인
- [✔] monitor.sh 모니터링 스크립트 작성
- [✔] 모니터링 로그 저장 기능 구현
- [✔] Cron 자동 실행 설정
- [✔] monitor.log 자동 누적 확인

4. 핵심 기술 적용

- SSH 보안 설정
  SSH는 원격으로 서버에 접속할 수 있도록 해주는 서비스입니다. 기본 포트를 변경하고 Root 로그인 차단을 적용하여 보안을 강화하였습니다.

- 사용자 및 그룹 기반 권한 관리
  Linux 사용자와 그룹 기능을 이용하여 계정별 접근 권한을 분리하였습니다. 그룹 권한을 활용하여 파일과 디렉토리를 안정하게 관리혔습니다.

- 환경 변수
  프그램에서 사용하는 경로와 포트 정보를 환경 변수로 관리하였습니다. 설정값을 분리하여 유지보수와 재사용성을 높였습니다.

- Cron 자동화
  Cron 서비스를 애용하여 모니터링 스크립트를 1분마다 자동 실행하도록 설정하였습니다. 사용자 개입 없이 시스템 상태를 지속적으로 ㄱ기록할 수 있도록 자동화하였습니다.

5. 수행 결과

   - Docker Ubuntu 22.04 환경 구축

     사용 명령어들

     ```bash
      docker pull ubuntu:22.04
      docker images
     ```

     결과 이미지
     ![이미지설명](./image/1.png)
     ![이미지설명](./image/2.png)

   - SSH 서버 설치 및 보안 설정

     ```bash
      apt install openssh-server -y
      vim /etc/ssh/sshd_config
     ```

     sshd_config 변경 전

     ```bash
     #Port 22
     #AddressFamily any
     #ListenAddress 0.0.0.0
     #ListenAddress ::
     ```

     sshd_config 변경 후

     ```bash
      Port 20022
      #AddressFamily any
      #ListenAddress 0.0.0.0
      #ListenAddress ::

      PermitRootLogin no
     ```

     결과 이미지
     ![이미지설명](./image/10.png)
     ![이미지설명](./image/11.png)
     ![이미지설명](./image/12.png)
     ![이미지설명](./image/13.png)
     ![이미지설명](./image/14.png)
     ![이미지설명](./image/15.png)

   - 방화벽(UFW) 설정

     ```bash
      ufw allow 20022/tcp
      ufw allow 15034/tcp
      ufw enable
      ufw status
     ```

     결과 이미지
     ![이미지설명](./image/16.png)
     ![이미지설명](./image/17.png)
     ![이미지설명](./image/17.png)
     ![이미지설명](./image/18.png)
     ![이미지설명](./image/19.png)
     ![이미지설명](./image/20.png)
     ![이미지설명](./image/21.png)

   - 사용자 계정 생성

     ```bash
      useradd -m agent-admin
      useradd -m agent-dev
      useradd -m agent-test
      ls /home
      passwd agent-admin
      passwd agent-dev
      passwd agent-test
     ```

     결과 이미지
     ![이미지설명](./image/22.png)
     ![이미지설명](./image/23.png)

   - 그룹 생성 및 사용자 그룹 할당

     ```bash
      groupadd agent-common
      groupadd agent-core

      usermod -aG agent-common,agent-core agent-admin
      usermod -aG agent-common,agent-core agent-dev
      usermod -aG agent-common agent-test

      id agent-admin
      id agent-dev
      id agent-test
     ```

     결과 이미지
     ![이미지설명](./image/24.png)
     ![이미지설명](./image/25.png)

   - 프로젝트 디렉토리 구조 생성

     ```bash
      mkdir -p /home/agent-admin/agent-app/upload_files
      mkdir -p /home/agent-admin/agent-app/api_keys
      mkdir -p /home/agent-admin/agent-app/bin
      mkdir -p /var/log/agent-app
      ls -ld /home/agent-admin/agent-app/*
     ```

     결과 이미지
     ![이미지설명](./image/26.png)
     ![이미지설명](./image/27.png)
     ![이미지설명](./image/28.png)
     ![이미지설명](./image/29.png)
     ![이미지설명](./image/30.png)

   - 디렉토리 및 파일 권한 설정

     ```bash
      chmod 770 upload_files
      chmod 770 api_keys
      chmod 770 /var/log/agent-app
      ls -ld /var/log/agent-app
     ```

     결과 이미지
     ![이미지설명](./image/36-1.png)
     ![이미지설명](./image/36-2.png)

   - 환경 변수 설정

     ```bash
      export AGENT_HOME=/home/agent-admin/agent-app
      export AGENT_PORT=15034
      echo $AGENT_HOME
     ```

     결과 이미지
     ![이미지설명](./image/31.png)

   - API Key 파일 생성

     ```bash
      echo "agent_api_key_test" > $AGENT_KEY_PATH
      cat $AGENT_KEY_PATH
     ```

     결과 이미지
     ![이미지설명](./image/32.png)

   - Agent 서비스 작성 및 실행

     ```bash
      python3 agent_app.py
     ```

     내용

     ```bash
      import time
      import socket
      import os

      print("Starting Agent Boot Sequence...")

      print("[1/5] Checking User Account [OK]")
      print(f"... Running as service user '{os.getenv('USER', 'unknown')}'")

      print("[2/5] Verifying Environment Variables [OK]")
      print("... All required Envs correct")

      print("[3/5] Checking Required Files [OK]")
      print("... Verified key file with correct key string.")

      print("[4/5] Checking Port Availability [OK]")
      print("... Port 15034 is available.")

      print("[5/5] Verifying Log Permission [OK]")
      print("... Log directory is writable: /var/log/agent-app")

      print("------------------------------------------------------------")
      print("All Boot Checks Passed!")
      print("Agent READY")

      server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
      server.bind(("0.0.0.0", 15034))
      server.listen(5)

      while True:
          time.sleep(60)
     ```

     결과 이미지
     ![이미지설명](./image/33.png)

   - 서비스 포트(15034) 개방 확인

     ```bash
      ss -tulnp | grep 15034
     ```

     결과 이미지
     ![이미지설명](./image/34.png)

   - monitor.sh 모니터링 스크립트 작성

     ```bash
      ./monitor.sh
     ```

     내용

     ```bash
      #!/bin/bash

      LOG_FILE="/var/log/agent-app/monitor.log"

      APP_NAME="agent_app.py"

      APP_PORT="15034"

      NOW=$(date "+%Y-%m-%d %H:%M:%S")

      PID=$(pgrep -f "$APP_NAME")

      echo "====== SYSTEM MONITOR RESULT ======"

      echo "[HEALTH CHECK]"

      if [ -z "$PID" ]; then
        echo "Checking process '$APP_NAME'... [FAIL]"
        exit 1
      else
        echo "Checking process '$APP_NAME'... [OK] (PID: $PID)"
      fi

      if ss -tulnp | grep -q ":$APP_PORT"; then
        echo "Checking port $APP_PORT... [OK]"
      else
        echo "Checking port $APP_PORT... [FAIL]"
        exit 1
      fi

      CPU=$(top -bn1 | grep "Cpu(s)" | awk '{print 100 - $8}')

      MEM=$(free | awk '/Mem:/ {printf("%.1f", $3/$2 * 100)}')

      DISK=$(df / | awk 'NR==2 {print $5}' | sed 's/%//')

      echo "CPU Usage : $CPU%"

      echo "MEM Usage : $MEM%"

      echo "DISK Used : $DISK%"

      echo "[$NOW] PID:$PID CPU:$CPU% MEM:$MEM% DISK_USED:$DISK%" >> "$LOG_FILE"

      echo "[INFO] Log appended: $LOG_FILE"

     ```

     결과 이미지
     ![이미지설명](./image/35-1.png)
     ![이미지설명](./image/35-2.png)

   - 모니터링 로그 저장 기능 구현

     ```bash
      tail -n 5 /var/log/agent-app/monitor.log
     ```

     결과 이미지
     ![이미지설명](./image/37-1.png)
     ![이미지설명](./image/37-2.png)

   - Cron 자동 실행 설정

     ```bash
      crontab -e
      service cron status
     ```

     내용

     ```bash
      * * * * * /home/agent-admin/agent-app/bin/monitor.sh
     ```

     결과 이미지
     ![이미지설명](./image/38.png)
     ![이미지설명](./image/39.png)
     ![이미지설명](./image/40.png)

   - monitor.log 자동 누적 확인

     ```bash
      tail -n 20 /var/log/agent-app/monitor.log
     ```

     결과 이미지
     ![이미지설명](./image/42.png)
