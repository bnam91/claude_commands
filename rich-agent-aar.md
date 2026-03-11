daily task log를 읽고 Notion AAR(After Action Review) 페이지를 자동 생성하는 스킬이야.

## Notion DB
- **AAR DB ID**: `31c111a5778880eba30ad18811857baa` (AGENT LEVEL A)
- DB 속성: `이름` (title), `날짜` (date)

## 스프레드시트
- **시트 ID**: `1SHWP0U72-bTzK__nnTnmB3UOqCbWDG__E7WwN-l22J0`
- **탭**: DAY_START_HOUR=6 기준 오늘 날짜 (`YYYY-MM-DD`)
  - 새벽 6시 이전이면 전날 탭 사용

## 실행 순서

### 1. task log 조회
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
```

컬럼 구조: A=시작, B=예상종료, C=실제종료, D=한 일, E=상태, F=소요, G=코멘트, H=평가

### 2. 시간 집계 계산

F열(소요) 값을 파싱해서 분 단위로 환산 후 카테고리별 합산.
카테고리는 G열 코멘트의 🔴/🟡/🔵 이모지 또는 D열 내용으로 판단.

```python
import re

def parse_minutes(s):
    # "1시간 14분", "55분", "13시간 20분" → 분 단위 정수
    h = re.search(r'(\d+)시간', s)
    m = re.search(r'(\d+)분', s)
    return (int(h.group(1)) * 60 if h else 0) + (int(m.group(1)) if m else 0)

totals = {'🔴': 0, '🟡': 0, '🔵': 0, '😴': 0, '기타': 0}
for row in rows[1:]:  # 헤더 제외
    if len(row) < 6: continue
    minutes = parse_minutes(row[5])  # F열
    comment = row[6] if len(row) > 6 else ''
    task = row[3] if len(row) > 3 else ''
    
    if '🔴' in comment or '🔴' in task:
        totals['🔴'] += minutes
    elif '🔵' in comment or '🔵' in task:
        totals['🔵'] += minutes
    elif '🟡' in comment or '🟡' in task:
        totals['🟡'] += minutes
    elif any(k in task for k in ['수면', '기상', '휴식', '이동', '식사', '귀가', '샤워']):
        totals['😴'] += minutes
    else:
        totals['기타'] += minutes

total_all = sum(totals.values())
# 비율 계산: round(v / total_all * 100, 1)
```

### 3. AAR 분석 기준

**문제점 추출 기준**
- 🔵가 🔴보다 많은 시간 차지할 때
- 🔴 비중이 전체의 30% 미만일 때
- 새벽 1시 이후 신규 작업 시작
- 예상 시간 2배 이상 초과 반복
- 수면 리듬 붕괴 징후

**내일 우선 순위**
- 미완료 🔴 항목 먼저
- 이번주/이번달 로드맵 마감 임박 항목

### 4. 하루 점수 산정 (100점) — JSON으로 구조화 후 입력

분석 완료 후 아래 JSON을 먼저 생성한다. **이 JSON이 점수 callout의 단일 소스가 된다.**

```json
{
  "red_pct": 35,
  "scores": {
    "red": 28,
    "principle": 20,
    "roadmap": 12,
    "energy": 7
  },
  "total": 67,
  "grade": "보통",
  "color": "yellow"
}
```

**필드 정의**

| 필드 | 설명 |
|------|------|
| `red_pct` | 🔴 시간 / 전체 작업시간(수면 제외) × 100 (정수) |
| `scores.red` | 🔴 비중 점수 (0~40). 50%=40, 30%=24, 10%=8. 비례 계산 |
| `scores.principle` | 원칙 준수 (0~30). 새벽 1시 원칙(10) + 오전 콘텐츠 블록(10) + 🔵 2시간 상한(10) |
| `scores.roadmap` | 로드맵 진척 (0~20). 상=20, 중=12, 하=5 |
| `scores.energy` | 에너지 관리 (0~10). 수면 6시간+(5) + 리듬 붕괴 없음(5) |
| `total` | scores 합계 |
| `grade` | 80+=훌륭, 60~79=보통, 40~59=위험, 39이하=적신호 |
| `color` | 80+=green, 60~79=yellow, 40~59=orange, 39이하=red |

이 JSON을 기반으로 Notion callout 텍스트를 생성:
```
🏆 오늘 점수: {total}점 ({grade})
🔴 비중: {scores.red}점/40 ({red_pct}%) | 원칙: {scores.principle}점/30 | 로드맵: {scores.roadmap}점/20 | 에너지: {scores.energy}점/10
```

### 5. 반복 문제 감지 + 개선 아이디어

AAR DB의 최근 7개 페이지 문제점 섹션을 비교해서 2회 이상 반복 패턴 감지.

**반복 문제 → 개선 아이디어 매핑**

| 반복 패턴 | 개선 아이디어 (구조/환경 우선) |
|----------|------------------------------|
| 🔵가 🔴 압도 (2회+) | 🔵 시작 전 rich-agent 승인 필요 규칙 + 캘린더 🔴 블록 선점 |
| 새벽 1시 이후 작업 (2회+) | 자정 맥 알림 설정 / 이닦기→컴퓨터 닫기 루틴화 |
| 예상 2배 초과 반복 | 타이머 울리면 미완이어도 저장하고 다음날로 넘김 |
| 할일 정리 계속 밀림 | 기상 직후 5분 고정 루틴, 다른 작업 시작 전 조건으로 설정 |
| 수면 리듬 붕괴 반복 | 취침 시간 무관하게 기상 시간 7시 고정 알람 |
| 위임 가능 업무 직접 처리 | 미리알림 inbox에 "→ 지혜" 태그로 추가하는 습관 |

개선 아이디어는 **한 번에 1개만** 제안한다.

### 6. Notion 페이지 구조
```
## ⏱ 시간 분배 (테이블)
  분류 | 주요 업무 | 소요시간 | 비율
  🔴 핵심 | ... | X시간 Y분 | 35%
  🔵 개발  | ... | X시간 Y분 | 28%
  🟡 위임  | ... | X시간 Y분 | 10%
  😴 회복  | ... | X시간 Y분 | 27%
  ─────────────────────────────
  합계     |     | X시간 Y분 | 100%

## ✅ 잘한 것          ← 불릿 리스트
## ⚠️ 문제점           ← 번호 리스트
## 🔁 반복 문제 & 개선 아이디어  ← 반복 감지 시에만 표시
## 🎯 내일 반드시 할 것 ← 번호 리스트
## 🏆 오늘 점수: {total}점 ({grade})  ← callout (color: score_json.color)
   └ 🔴 비중: {scores.red}점/40 ({red_pct}%) | 원칙: {scores.principle}점/30 | 로드맵: {scores.roadmap}점/20 | 에너지: {scores.energy}점/10
📌 [콜아웃] 총평       ← orange_background
```

**⚠️ 점수 callout은 반드시 4단계에서 생성한 score JSON 값으로만 채운다. 직접 숫자를 서술하지 않는다.**

점수별 콜아웃 색상: 80+=green, 60~79=yellow, 40~59=orange, 39이하=red

### 7. Notion 페이지 생성
```bash
NOTION_KEY=$(grep NOTION_API_KEY ~/github/api_key/.env | cut -d= -f2 | tr -d '\r')

curl -s -X POST "https://api.notion.com/v1/pages" \
  -H "Authorization: Bearer $NOTION_KEY" \
  -H "Notion-Version: 2022-06-28" \
  -H "Content-Type: application/json" \
  -d '{
    "parent": { "database_id": "31c111a5778880eba30ad18811857baa" },
    "properties": {
      "이름": { "title": [{ "text": { "content": "AAR" } }] },
      "날짜": { "date": { "start": "YYYY-MM-DD" } },
      "점수": { "number": {total} }
    },
    "children": [ ... ]
  }'
```

**{total}은 4단계 score JSON의 total 값**을 그대로 사용한다.

## 사용자 요청 처리

- "오늘 마감 피드백 해줘" / "AAR 만들어줘" / "하루 정리해줘" → 위 순서대로 실행
- 날짜 지정: "3월 7일 AAR 만들어줘" → 해당 탭 조회 후 생성

결과는 한국어로 정리하고, 생성된 Notion URL 출력해줘.
