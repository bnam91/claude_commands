daily_task_log 시트에 작업을 기록하고 관리하는 스킬이야.
**모든 시트 쓰기는 `task_timer.py`를 통해서만 한다. 직접 시트 API로 쓰지 않는다.**

## 시트 정보

- **스프레드시트 ID**: `1SHWP0U72-bTzK__nnTnmB3UOqCbWDG__E7WwN-l22J0`
- **탭명**: DAY_START_HOUR=6 기준 오늘 날짜 (`YYYY-MM-DD`)
  - 새벽 6시 이전이면 전날 탭 사용 (`task_timer.py`가 자동 처리)
  - 탭 생성 + 헤더 추가도 `task_timer.py`가 자동 처리

## 컬럼 구조 (task_timer.py HEADER 기준)

| A: 시작 | B: 예상종료 | C: 실제종료 | D: 한 일 | E: 상태 | F: 소요 | G: 코멘트 | H: 평가 |
|---------|------------|------------|---------|---------|---------|----------|---------|
| HH:MM | HH:MM | HH:MM | 작업명 | 진행중/완료 | X분/X시간 Y분 | rich-agent 판단 | — |

## 작업 시작

```bash
python3 ~/Documents/claude_skills/os_manager/task_timer.py start "작업명" [알람간격분=15] &
```
→ 탭 없으면 자동 생성 + 헤더 추가 + A~F열 기록 + 백그라운드 타이머

## 작업 완료

```bash
python3 ~/Documents/claude_skills/os_manager/task_timer.py done 행번호 '{"han_il":"실제 한 일", "comment":"rich-agent 코멘트", "eval":"평가"}'
```
→ C:실제종료, D:한 일, E:완료, F:소요(자동계산), G:코멘트, H:평가 기록

## 타이머 상태 확인

```bash
python3 ~/Documents/claude_skills/os_manager/task_timer.py status
```
출력: `TIMER_RUNNING`, `ROW_NUM`, `TAB`, `ELAPSED`, `START_TIME`

## 오늘 작업 조회 (읽기 전용)

```python
import sys
from datetime import datetime, timedelta
sys.path.insert(0, str(__import__('pathlib').Path.home() / 'Documents/github_cloud/module_auth'))
from auth import get_credentials
from googleapiclient.discovery import build

creds = get_credentials()
sheets = build('sheets', 'v4', credentials=creds)
SHEET_ID = '1SHWP0U72-bTzK__nnTnmB3UOqCbWDG__E7WwN-l22J0'
now = datetime.now()
tab = (now - timedelta(days=1)).strftime('%Y-%m-%d') if now.hour < 6 else now.strftime('%Y-%m-%d')

result = sheets.spreadsheets().values().get(
    spreadsheetId=SHEET_ID, range=f'{tab}!A:H'
).execute()
rows = result.get('values', [])
print(f"탭: {tab}")
for i, r in enumerate(rows):
    print(f"[{i+1}] {r}")
```

## ⚠️ 직접 시트 쓰기 금지 사항

절대 하지 말 것:
- `sheets.values().update()` 로 직접 행 추가
- 탭을 `batchUpdate`로 직접 생성 (헤더 없이 생성됨)
- A열에 날짜(`YYYY-MM-DD`) 입력 (A열은 시작시간 `HH:MM` 전용)

## 사용자 요청 처리

| 사용자 말 | 처리 |
|---------|------|
| "작업 시작할게" / "시작할게" | `task_timer.py start "작업명" [분]` |
| "다했어" / "완료" | `status`로 행번호 확인 → `done 행번호 '{"han_il":"...", "comment":"...", "eval":"..."}'` |
| "타이머 어떻게 되고 있어?" | `status` |
| "오늘 작업 보여줘" | 위 조회 코드 실행 |
