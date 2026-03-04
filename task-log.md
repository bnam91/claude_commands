daily_task_log 시트에 작업을 기록하고 관리하는 스킬이야.

## 시트 정보

- **스프레드시트 ID**: `1SHWP0U72-bTzK__nnTnmB3UOqCbWDG__E7WwN-l22J0`
- **탭명**: 오늘 날짜 (`YYYY-MM-DD` 형식, 없으면 자동 생성)
- **드라이브 폴더**: `rich-agent_v0.1.0`

## 컬럼 구조

| A: 날짜 | B: 시작시간 | C: 작업 | D: 상태 | E: 완료시간 | F: 소요시간 | G: 코멘트 |
|---------|------------|---------|---------|------------|------------|----------|
| YYYY-MM-DD | HH:MM | 작업명 | 진행중/완료/보류 | HH:MM | X분 Y초 | Claude 코멘트 |

## 공통 헬퍼 (Python)

```python
import sys
from datetime import datetime
sys.path.insert(0, str(__import__('pathlib').Path.home() / 'Documents/github_cloud/module_auth'))
from auth import get_credentials
from googleapiclient.discovery import build

creds = get_credentials()
sheets = build('sheets', 'v4', credentials=creds)
SHEET_ID = '1SHWP0U72-bTzK__nnTnmB3UOqCbWDG__E7WwN-l22J0'
TAB = datetime.now().strftime('%Y-%m-%d')  # 오늘 날짜 탭

def ensure_tab():
    meta = sheets.spreadsheets().get(spreadsheetId=SHEET_ID).execute()
    existing = [s['properties']['title'] for s in meta.get('sheets', [])]
    if TAB not in existing:
        sheets.spreadsheets().batchUpdate(
            spreadsheetId=SHEET_ID,
            body={'requests': [{'addSheet': {'properties': {'title': TAB}}}]}
        ).execute()
```

## 작업 추가

```python
ensure_tab()
result = sheets.spreadsheets().values().get(
    spreadsheetId=SHEET_ID, range=f'{TAB}!A:A'
).execute()
next_row = len(result.get('values', [])) + 1
now = datetime.now()

sheets.spreadsheets().values().update(
    spreadsheetId=SHEET_ID,
    range=f'{TAB}!A{next_row}',
    valueInputOption='USER_ENTERED',
    body={'values': [[
        now.strftime('%Y-%m-%d'),
        now.strftime('%H:%M'),
        '작업명',   # ← 변경
        '진행중',
        '', '', ''
    ]]}
).execute()
print(f'{next_row}행 추가 완료 (탭: {TAB})')
```

## 완료 처리 (소요시간 + 코멘트 포함)

```python
# row_num = 완료할 행, elapsed_str = "12분 30초", comment = Claude 코멘트
done_time = datetime.now().strftime('%H:%M')
sheets.spreadsheets().values().update(
    spreadsheetId=SHEET_ID,
    range=f'{TAB}!D{row_num}:G{row_num}',
    valueInputOption='USER_ENTERED',
    body={'values': [['완료', done_time, elapsed_str, comment]]}
).execute()
```

## 오늘 작업 조회

```python
result = sheets.spreadsheets().values().get(
    spreadsheetId=SHEET_ID, range=f'{TAB}!A:G'
).execute()
rows = result.get('values', [])
for i, r in enumerate(rows):
    if not r:
        continue
    status = r[3] if len(r) > 3 else ''
    elapsed = r[5] if len(r) > 5 else ''
    comment = r[6] if len(r) > 6 else ''
    print(f"[{i+1}행] {r[2]} | {status} | {elapsed} | {comment}")
```

## task_timer.py 연동

```bash
# 작업 시작 (시트 기록 + 백그라운드 타이머)
python3 ~/Documents/claude_skills/os_manager/task_timer.py start "작업명" {row_num} 5 &

# 완료 처리 (소요시간 자동 계산 + 코멘트 입력)
python3 ~/Documents/claude_skills/os_manager/task_timer.py done {row_num} "Claude 코멘트"

# 현재 타이머 상태
python3 ~/Documents/claude_skills/os_manager/task_timer.py status
```

## 사용자 요청 처리

- "작업 기록해줘" / "시트에 남겨줘" → 오늘 탭에 작업 추가
- "다했어" → status로 소요시간 확인 → 코멘트 생성 → done 실행
- "오늘 작업 보여줘" → 오늘 탭 전체 조회
- "타이머 어떻게 되고 있어?" → status 실행
- rich-agent와 연동: 작업 배정 시 자동 시작, 완료 보고 시 자동 종료
