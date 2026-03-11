맥 노트(Notes) 앱을 제어하는 스킬이야.
스크립트 경로: ~/Documents/claude_skills/app_notes_control/app_notes_control.py

## 주요 명령

### 폴더 목록
```bash
python3 ~/Documents/claude_skills/app_notes_control/app_notes_control.py --list-folders
```

### 노트 목록 조회
```bash
# 전체 (최근 20개)
python3 ~/Documents/claude_skills/app_notes_control/app_notes_control.py --list

# 폴더 지정
python3 ~/Documents/claude_skills/app_notes_control/app_notes_control.py --list --folder "폴더명"

# 개수 지정
python3 ~/Documents/claude_skills/app_notes_control/app_notes_control.py --list --limit 50
```

### 노트 읽기
```bash
python3 ~/Documents/claude_skills/app_notes_control/app_notes_control.py --read --title "노트제목"
```

### 노트 생성
```bash
python3 ~/Documents/claude_skills/app_notes_control/app_notes_control.py \
  --create --title "제목" --body "내용" [--folder "폴더명"]
```

### 노트에 내용 추가 (append)
```bash
python3 ~/Documents/claude_skills/app_notes_control/app_notes_control.py \
  --append --title "제목" --body "추가할 내용"
```

### 노트 삭제
```bash
python3 ~/Documents/claude_skills/app_notes_control/app_notes_control.py --delete --title "제목"
```

### 키워드 검색
```bash
python3 ~/Documents/claude_skills/app_notes_control/app_notes_control.py --search "키워드"
```

## 사용자 요청 처리

- "노트 목록 보여줘" → `--list` 실행
- "XX 노트 읽어줘" → `--read --title "XX"` 실행
- "XX 노트 만들어줘" → `--create` 실행
- "XX 노트에 내용 추가해줘" → `--append` 실행
- "XX 노트 지워줘" → `--delete` 실행
- "XX 검색해줘" → `--search` 실행
- rich-agent 연동: daily 브리핑, PM 회의 결과 자동 기록 시 활용

결과는 한국어로 정리해서 보여줘.
