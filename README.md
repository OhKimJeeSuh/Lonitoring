# 🖨️Lonitoring

<br>

## 목차
1. [📬 프로젝트 개요](#1--프로젝트-개요)  
2. [🎓 팀원소개](#2--팀원소개) 
3. [⚙️ 개발 환경](#3-%EF%B8%8F-개발-환경)   
4. [🌟 주요 기능](#4--주요-기능)  
　      4-1. [서버 리소스 모니터링](#4-1-서버-리소스-모니터링)  
　      4-2. [보안 이벤트 모니터링](#4-2-보안-이벤트-모니터링)  
　      4-3. [cron 활용](#4-3-cron-활용)  
5. [🖥️ 실행](#5-%EF%B8%8F-실행)  
　      5-1. [서버 리소스](#5-1-서버-리소스)  
　      5-2. [로그인 실패 기록](#5-2-로그인-실패-기록)  
　      5-3. [결과 화면](#5-3-결과-화면)  
6. [💣 트러블 슈팅](#6--트러블-슈팅)  
7. [💡고찰](#7--고찰) 
<br><br>


## 1. 📬 프로젝트 개요
 이 프로젝트는 서버의 리소스 사용량 및 보안 이벤트를 수집, 저장하여 인프라를 효율적으로 관리하는 시스템을 구축한다. 리소스는 CPU, 메모리, 디스크 사용량을, 보안 이벤트는 로그인 실패를 수집하며, cron을 활용해 일정 주기로 해당 데이터를 log로 기록하고 분석한다.<br>
 이를 통해 리눅스 명령어 및 시스템 관리 스킬을 학습하며, 서버의 성능 최적화 및 리눅스 시스템 관리  및 보안 운영에 대한 인사이트를 향상시키고, 로그 분석을 통한 문제 해결 역량을 강화하는 것을 목표로 한다.

<br><br>

## 2. 🎓 팀원소개

|<img src="https://avatars.githubusercontent.com/u/114637614?v=4" width="150" height="150"/>|<img src="https://avatars.githubusercontent.com/u/165532198?v=4" width="150" height="150"/>|<img src="https://avatars.githubusercontent.com/u/193404366?v=4" width="150" height="150"/>|<img src="https://avatars.githubusercontent.com/u/79669001?v=4" width="150" height="150"/>|
|:-:|:-:|:-:|:-:|
|오현두<br/>[@HyunDooBoo](https://github.com/HyunDooBoo)|김소연<br/>[@ssoyeonni](https://github.com/ssoyeonni)|지수근<br/>[@SuGeunJee](https://github.com/SuGeunJee)|서소원<br/>[@PleaseErwin](https://github.com/PleaseErwin)|

<br><br>


## 3. ⚙️ 개발 환경

| 항목               | 상세 내용                |
|--------------------|-------------------------|
| **가상화 환경**    | VMware                  |
| **VM 운영체제**    | Ubuntu 24.04.2 LTS      |
| **원격 접속 도구** | MobaXterm               |
| **스크립트 실행 환경** | Bash Shell           |
| **로그 저장 위치** | `/var/log/` 디렉토리    |

<br><br>

## 4. 🌟 주요 기능
### 4-1. 서버 리소스 모니터링
- CPU, 메모리, 디스크 사용량을 일정 주기로 기록
- 리눅스 명령어: top, free, df 활용

<br>

### 4-2. 보안 이벤트 모니터링
- SSH 로그인 실패 기록
- 리눅스 명령어: grep "Failed password”

<br>

### 4-3. cron 활용
- 1초 주기로 리소스 데이터를 수집하여 cpu_info.log에 저장
- 1분 주기로 로그인 실패 기록을 auth.log에서 가져와 failed_login/현재 날짜.log에 저장

<br><br>

## 5. 🖥️ 실행
### 5-1. 서버 리소스
**리소스 사용량 저장 스크립트**
> 1초에 한 번씩 CPU 사용량, CPU 코어 수, 메모리 사용량, CPU 부하, 디스크 사용량을 출력하여 저장하는 스크립트
![image](https://github.com/user-attachments/assets/39ae9fd0-2d30-4b0c-b324-eb8d80ccc3d0)



```
#!/bin/bash

# 로그 파일 경로
LOG_FILE="/var/log/cpu_info.log"

#반복 횟수 설정 (60번)
MAX_COUNT=60

#반복 변수
COUNT=0


# 반복문
while [ $COUNT -lt $MAX_COUNT ]
do
    # 현재 시간
    TIMESTAMP=$(date "+%Y-%m-%d %H:%M:%S")

    # CPU 사용률
    CPU_USAGE=$(top -bn1 | grep "Cpu(s)" | awk '{print $2 + $4}')

    # CPU 코어 수
    CPU_CORES=$(nproc)

    # 메모리 사용량
    MEMORY_USAGE=$(free -h | grep Mem | awk '{print $3 "/" $2}')

    # uptime (로드 평균)
    LOAD_AVG=$(uptime | awk -F'load average:' '{print $2}')

    # 디스크 사용량 (루트 디렉토리)
    DISK_USAGE=$(df -h / | awk 'NR==2 {print $5}')

    # 로그 기록
    echo "$TIMESTAMP | CPU 사용률: $CPU_USAGE% | 코어 수: $CPU_CORES | 로드 평균: $LOAD_AVG | 디스크 사용량: $DISK_USAGE | 메모리 사용량 : $MEMORY_USAGE" >> $LOG_FILE

    # 1초 대기
    sleep 1

    # 반복 횟수 증가
    COUNT=$((COUNT + 1))
done
```

**리소스 기록 cron 설정**
```
* * * * * /root/linux_practice/cpu_info.sh
```

<br>

### 5-2. CPU 사용량 감지 및 종료
**CPU 사용량 감지 및 종료 스크립트**
> 5초마다 CPU 사용량을 체크해 현재 사용량에 따른 메세지를 출력 및 80퍼 초과시 사용량이 가장 높은 프로그램 강제 종료하는 스크립트 <br>
![image](https://github.com/user-attachments/assets/081e0161-1dc8-4969-9494-4fe7212e822c)


```
#!/bin/bash

LOG_FILE="/var/log/cpu_usage_monitor.log"

# CPU 사용량 임계값 (80%와 90%)
CPU_NORMAL_THRESHOLD=70
CPU_WARNING_THRESHOLD=70
CPU_CRITICAL_THRESHOLD=90
COUNT=0
MAX_COUNT=11

while [ $COUNT -lt $MAX_COUNT ];
do
    # 현재 CPU 사용량 (top 명령어로 추출)
    CPU_USAGE=$(top -bn1 | grep "Cpu(s)" | awk '{print $2 + $4}')

    # CPU 사용량이 70% 미만일 때 알림
    if (( $(echo "$CPU_USAGE <= $CPU_NORMAL_THRESHOLD" | bc -l) )); then
        echo "$(date) - CPU 사용량이 정상적입니다." >> $LOG_FILE
    fi

    # CPU 사용량이 70% 초과일 때 경고
    if (( $(echo "$CPU_USAGE > $CPU_WARNING_THRESHOLD" | bc -l) )); then
        # 경고 메시지 출력 (80% 초과)
        echo "$(date) - CPU 사용량이 $CPU_USAGE%를 초과했습니다! (경고)" >> $LOG_FILE
        echo "" >> $LOG_FILE

        # CPU를 많이 사용하는 프로그램 찾기 (상위 5개)
        echo "$(date) - CPU를 많이 사용하는 프로그램:" >> $LOG_FILE
        echo "" >> $LOG_FILE
        ps aux --sort=-%cpu | head -n 6 >> $LOG_FILE
    fi

    # CPU 사용량이 80% 초과일 때 정말 위험 경고 및 종료 안내
    if (( $(echo "$CPU_USAGE > $CPU_CRITICAL_THRESHOLD" | bc -l) )); then
        # (90% 초과)
        echo "$(date) - CPU 사용량이 $CPU_USAGE%를 초과했습니다!" >> $LOG_FILE
        echo "" >> $LOG_FILE

        # CPU를 많이 사용하는 프로그램 찾기 (상위 5개)
        echo "$(date) - CPU를 많이 사용하는 프로그램:" >> $LOG_FILE
        echo "" >> $LOG_FILE
        ps aux --sort=-%cpu | head -n 6 >> $LOG_FILE

        # 종료해야 할 프로세스를 추천하는 메시지
        echo "$(date) - CPU 사용량을 많이 차지하는 프로그램을 종료하세요." >> $LOG_FILE
        echo "" >> $LOG_FILE
  
        # 가장 많이 사용하는 프로그램 종료 (PID로 프로세스를 종료)
        PID_TO_KILL=$(ps aux --sort=-%cpu | awk 'NR==2 {print $2}')  # 2번째로 CPU 많이 사용하는 프로세스의 PID
        kill -9 $PID_TO_KILL
        echo "$(date) - CPU를 많이 차지하는 프로세스($PID_TO_KILL)가 종료되었습니
다." >> $LOG_FILE
        echo "" >> $LOG_FILE
    fi

    # 5초 대기
    sleep 5

    #반복 횟수 증가
    COUNT=$((COUNT + 1))
done
```
**로그인 성공 기록 cron 설정**
```
* * * * * /root/linux_practice/cpu_status.sh
```

<br>

### 5-2. 로그인 실패 기록
**로그인 실패 저장 스크립트**
> auth.log에 저장된 “Failed password” 로그를 받아와 로그인 실패 기록만 저장된 새로운 log파일을 생성하는 스크립트

```
#!/bin/bash

# 설정
LOG_DIR="/home/ubuntu/failed_login"
TODAY=$(date +%y%m%d)
LOG_FILE="$LOG_DIR/$TODAY.log"

# 디렉토리가 없으면 생성
mkdir -p "$LOG_DIR"

# 오늘 날짜의 타임스탬프 패턴 (2025-03-18 형식)
TODAY_DATE_PATTERN=$(date +"%Y-%m-%d")

# auth.log에서 오늘 날짜의 Failed password 로그만 추출
grep "Failed password" /var/log/auth.log | grep "$TODAY_DATE_PATTERN" > "$LOG_FILE.today"

# 현재 로그 파일이 있으면 중복 제거를 위해 비교
if [ -f "$LOG_FILE" ]; then
  # 새 로그에서 기존 로그에 없는 항목만 추출
  comm -13 <(sort "$LOG_FILE") <(sort "$LOG_FILE.today") > "$LOG_FILE.new"
else
  # 파일이 없으면 모든 오늘 로그를 새 로그로 간주
  cp "$LOG_FILE.today" "$LOG_FILE.new"
fi

# 새 로그가 있으면 추가
if [ -s "$LOG_FILE.new" ]; then
  cat "$LOG_FILE.new" >> "$LOG_FILE"
  echo "$(date '+%Y-%m-%d %H:%M:%S') - 새 로그 $(wc -l < "$LOG_FILE.new")개 추가됨" >> "$LOG_DIR/monitor.log"
else
  echo "$(date '+%Y-%m-%d %H:%M:%S') - 새 로그 없음" >> "$LOG_DIR/monitor.log"
fi

# 임시 파일 정리
rm -f "$LOG_FILE.today" "$LOG_FILE.new"
```

**로그인 실패 기록 cron 설정**
```
* * * * * /home/ubuntu/failed_login/failed_login_monitor.sh
```

<br>

### 5-3. 결과 화면

**로그인 실패 기록 결과 화면**
- 현재 로그 기록
  ![Image](https://github.com/user-attachments/assets/d533333e-b889-4d4c-82f8-22c6838b37be) <br>

- 현 시간보다 뒤의 log 발생 시 기록 → 1분 단위로 모니터링
  ![Image](https://github.com/user-attachments/assets/25da3d0a-6367-4cb5-bdd4-36fbecd52282) <br>

- 추가된 로그 기록
  ![Image](https://github.com/user-attachments/assets/6b4e0901-a9a5-4a89-868e-ad9131b07434) <br>

<br><br>

## 6. 💣 트러블 슈팅

<br><br>

## 7. 💡 고찰

<br><br>
