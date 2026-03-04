현재 OS에 등록된 예약 작업(cron, launchd, Task Scheduler)을 조회하는 스킬이야.

## macOS

### crontab (사용자)
```bash
crontab -l
```

### launchd agents (사용자 등록)
```bash
ls ~/Library/LaunchAgents/
launchctl list | grep -v "^-"
```

### launchd agents/daemons (시스템)
```bash
ls /Library/LaunchAgents/
ls /Library/LaunchDaemons/
```

### 특정 job 상세 보기
```bash
cat ~/Library/LaunchAgents/파일명.plist
```

### plist에서 스케줄 정보 읽기
- `StartCalendarInterval` → 특정 시각 반복 (Hour/Minute/Weekday 등)
- `StartInterval` → N초마다 반복
- `RunAtLoad` → 로그인 시 1회 실행
- `StartOnMount` → 디스크 마운트 시 실행

crontab 표현식 해석:
```
* * * * *
│ │ │ │ └ 요일 (0=일, 6=토)
│ │ │ └── 월
│ │ └──── 일
│ └────── 시
└──────── 분
```

---

## Windows

### 전체 예약 작업 목록
```cmd
schtasks /query /fo LIST /v
```

### 간략히 보기
```cmd
schtasks /query /fo TABLE
```

### 특정 작업 상세
```cmd
schtasks /query /tn "작업명" /fo LIST /v
```

---

## 등록된 사용자 스크립트 (os_manager)

| 스크립트 | 경로 | 설명 |
|----------|------|------|
| trash_today_downloads.py | `~/Documents/claude_skills/os_manager/trash_today_downloads.py` | 오늘 다운로드된 파일을 휴지통으로 이동 |

예약 등록 시 예시:
```bash
# 매일 자정에 실행
0 0 * * * /usr/bin/python3 ~/Documents/claude_skills/os_manager/trash_today_downloads.py
```

---

## 사용자 요청 처리

- "예약 확인해줘" / "크론 목록" → OS 감지 후 해당 명령어 실행
- macOS면 crontab -l + launchctl list 실행
- Windows면 schtasks /query 실행
- 결과 한국어로 정리해서 보여줘
- 아래 패턴은 앱/시스템 자동 등록 항목이므로 **제외**하고 보여줘:
  - com.apple.*, com.google.*, com.microsoft.*, com.openai.*, com.adobe.*
  - 앱 이름이 포함된 것 (keystone, updater, helper, agent 등 키워드)
- crontab은 전부 사용자 등록으로 간주
- LaunchAgents 중 위 패턴에 해당하지 않는 것만 표시
- 직접 등록한 항목이 없으면 "등록된 사용자 예약 작업 없음" 으로 표시
- 각 항목마다 아래 정보를 함께 표시:
  - **실행 주기**: 매일/매주/N분마다/로그인 시 1회 등 사람이 읽기 쉬운 형태로
  - **1회성 여부**: RunAtLoad만 있고 반복 트리거 없으면 "1회성(로그인 시)", StartInterval/StartCalendarInterval 있으면 "반복"
  - **다음 실행 예정 시각** (계산 가능한 경우)
