맥 미리알림(Reminders) 앱의 **GG_timeline** 목록을 제어하는 스킬이야.
스크립트 경로: ~/Documents/github_skills/app_reminder_control/app_reminders_control.py
기본 목록: GG_timeline (번호: 4)

## 사용 가능한 기능

### 1. GG_timeline 전체 조회
```bash
python3 ~/Documents/github_skills/app_reminder_control/app_reminders_control.py "GG_timeline"
```

### 2. 특정 섹션 조회
전체 조회 후 해당 섹션만 필터링해서 보여줘.
섹션 예시: 1주차, 2주차, 3주차, 런칭 이후

### 3. 미리알림 추가 (섹션 지정)
```bash
python3 ~/Documents/github_skills/app_reminder_control/app_reminders_control.py 4 "섹션이름, 항목제목"
```

### 4. 우선순위 변경
EventKit으로 직접 처리:
```python
import sys, time
sys.path.insert(0, '~/Documents/github_skills/app_reminder_control')
from app_reminders_control import _get_event_store, fetch_reminders_sync

event_store = _get_event_store()
calendars = event_store.calendarsForEntityType_(1)
cal = next((c for c in calendars if c.title() == 'GG_timeline'), None)
predicate = event_store.predicateForRemindersInCalendars_([cal])
reminders = fetch_reminders_sync(event_store, predicate)

# 우선순위: 1=높음, 5=중간, 9=낮음, 0=없음
for r in reminders:
    if r.title() in targets:
        r.setPriority_(1)
        event_store.saveReminder_commit_error_(r, True, None)
```

## 실행 방법

사용자의 요청을 분석하여 GG_timeline 목록에 대해 Bash 도구로 실행해줘.

- "전체 보여줘" → GG_timeline 전체 조회
- "1주차 보여줘" → 전체 조회 후 1주차 섹션만 필터링 출력
- "1주차에 XX 추가해줘" → 1주차 섹션에 항목 추가
- "XX 우선순위 높음으로" → EventKit으로 우선순위 변경

결과는 한국어로 정리해서 보여줘.
