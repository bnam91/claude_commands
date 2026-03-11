맥 미리알림(Reminders) 앱을 제어하는 스킬이야.
스크립트 경로: ~/Documents/claude_skills/app_reminder_control/app_reminders_control.py

## 사용 가능한 기능

### 1. 목록 이름만 빠르게 확인
```
python ~/Documents/claude_skills/app_reminder_control/app_reminders_control.py --lists
```
단, 이 옵션은 없으므로 아래 방식으로 대체:
Bash로 스크립트를 import해서 `get_list_names()` 함수 호출, 또는 아래 전체 목록 방식 사용

### 2. 전체 목록 확인 (번호 + 이름)
스크립트를 실행하면 목록이 표시됨. 번호 없이 실행 시 대화형 모드이므로, 반드시 번호나 이름으로 직접 지정해야 함.

목록 이름 확인용 (Python 한 줄):
```bash
python3 -c "
import sys; sys.path.insert(0, '~/Documents/claude_skills/app_reminder_control')
from app_reminders_control import get_list_names
names = get_list_names()
for i, n in enumerate(names, 1): print(f'{i}. {n}')
"
```

### 3. 특정 목록의 미리알림 조회
```bash
python ~/Documents/claude_skills/app_reminder_control/app_reminders_control.py [번호 또는 목록이름]
```
예시:
- `python ... 1` → 1번 목록 조회
- `python ... "군자"` → "군자" 포함 목록 조회

### 4. 미리알림 추가 (섹션 지정)
```bash
python ~/Documents/claude_skills/app_reminder_control/app_reminders_control.py [목록번호] "섹션이름, 항목제목"
```
예시:
- `python ... 4 "1주차, 명함만들기"` → 4번 목록의 "1주차" 섹션에 "명함만들기" 추가

### 5. 우선순위 변경
스크립트에 기능 없으므로 EventKit으로 직접 처리:
```python
import sys, time
sys.path.insert(0, '~/Documents/claude_skills/app_reminder_control')
from app_reminders_control import _get_event_store, fetch_reminders_sync

event_store = _get_event_store()
calendars = event_store.calendarsForEntityType_(1)
cal = next((c for c in calendars if c.title() == '목록이름'), None)
predicate = event_store.predicateForRemindersInCalendars_([cal])
reminders = fetch_reminders_sync(event_store, predicate)

targets = {'항목1', '항목2'}  # 변경할 항목 제목들
# 우선순위: 1=높음, 5=중간, 9=낮음, 0=없음
for r in reminders:
    if r.title() in targets:
        r.setPriority_(1)
        event_store.saveReminder_commit_error_(r, True, None)
```

## 실행 방법

사용자의 요청을 분석하여 적절한 명령을 Bash 도구로 실행해줘.

- "미리알림 목록 보여줘" → 목록 이름 조회 실행
- "XX 목록 확인해줘" → 해당 목록의 미리알림 전체 조회
- "XX에 YY 추가해줘" → 목록과 섹션 파악 후 추가 실행
- 목록 이름이 불확실하면 먼저 목록 이름 조회 후 매칭

결과는 한국어로 정리해서 보여줘.
