Google Sheets를 읽고, 쓰고, 수정하는 스킬이야.
스크립트 경로: ~/Documents/claude_skills/sheet_manager/sheet_manager.py

## 사용 가능한 기능

### 1. 탭 목록 확인
```bash
python3 ~/Documents/claude_skills/sheet_manager/sheet_manager.py tabs <spreadsheet_id>
```

### 2. 시트 읽기
```bash
# 전체 읽기
python3 ~/Documents/claude_skills/sheet_manager/sheet_manager.py read <spreadsheet_id> --tab <탭이름>

# 범위 지정 읽기
python3 ~/Documents/claude_skills/sheet_manager/sheet_manager.py read <spreadsheet_id> --tab <탭이름> --range A1:D10
```

### 3. 값 덮어쓰기
```bash
python3 ~/Documents/claude_skills/sheet_manager/sheet_manager.py write <spreadsheet_id> --tab <탭이름> --range A2:C2 --values '[["값1","값2","값3"]]'
```

### 4. 마지막 행에 추가
```bash
python3 ~/Documents/claude_skills/sheet_manager/sheet_manager.py append <spreadsheet_id> --tab <탭이름> --values '[["값1","값2","값3"]]'
```

### 5. 범위 삭제
```bash
python3 ~/Documents/claude_skills/sheet_manager/sheet_manager.py clear <spreadsheet_id> --tab <탭이름> --range A2:D10
```

## 사용 방법

사용자의 요청을 분석하여 적절한 명령을 Bash 도구로 실행해줘.

- "시트 읽어줘" → read 실행
- "값 입력해줘 / 수정해줘" → write 실행
- "행 추가해줘" → append 실행
- "탭 목록 보여줘" → tabs 실행
- "삭제해줘" → clear 실행

스프레드시트 ID는 URL에서 추출:
`https://docs.google.com/spreadsheets/d/<여기가_ID>/edit`

결과는 한국어로 정리해서 보여줘.
