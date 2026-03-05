MongoDB Atlas를 제어하는 스킬이야. DB/컬렉션 조회, 읽기, 검색, 추가, 수정, 삭제를 할 수 있어.
스킬 경로: ~/Documents/claude_skills/mongo_manager

---

## 명령어

### DB 목록 조회
```bash
cd ~/Documents/claude_skills/mongo_manager && python3 mongo_manager.py list-dbs
```

### 컬렉션 목록 조회
```bash
cd ~/Documents/claude_skills/mongo_manager && python3 mongo_manager.py list-cols --db DB명
```

### 데이터 읽기
```bash
# 전체 읽기
cd ~/Documents/claude_skills/mongo_manager && python3 mongo_manager.py read --db DB명 --col 컬렉션명

# 최근 N개 (내림차순)
cd ~/Documents/claude_skills/mongo_manager && python3 mongo_manager.py read --db DB명 --col 컬렉션명 --sort -_id --limit 10

# 필터 + 정렬
cd ~/Documents/claude_skills/mongo_manager && python3 mongo_manager.py read --db DB명 --col 컬렉션명 --filter '{"keyword":"라면"}' --sort -views --limit 20
```

### 텍스트 검색
```bash
cd ~/Documents/claude_skills/mongo_manager && python3 mongo_manager.py search --db DB명 --col 컬렉션명 --field keyword --query 라면 --limit 10
```

### 도큐먼트 추가
```bash
# 단일
cd ~/Documents/claude_skills/mongo_manager && python3 mongo_manager.py insert --db DB명 --col 컬렉션명 --data '{"key":"value"}'

# 복수 (배열)
cd ~/Documents/claude_skills/mongo_manager && python3 mongo_manager.py insert --db DB명 --col 컬렉션명 --data '[{"key":"v1"},{"key":"v2"}]'
```

### 도큐먼트 수정
```bash
# 단일
cd ~/Documents/claude_skills/mongo_manager && python3 mongo_manager.py update --db DB명 --col 컬렉션명 --filter '{"keyword":"라면"}' --set '{"views":"999"}'

# 다중 (--many)
cd ~/Documents/claude_skills/mongo_manager && python3 mongo_manager.py update --db DB명 --col 컬렉션명 --filter '{"duration":"SHORTS"}' --set '{"type":"short"}' --many
```

### 도큐먼트 삭제
```bash
# ID로 삭제
cd ~/Documents/claude_skills/mongo_manager && python3 mongo_manager.py delete --db DB명 --col 컬렉션명 --id 67b540744416eb7c2ce07392

# 필터로 삭제
cd ~/Documents/claude_skills/mongo_manager && python3 mongo_manager.py delete --db DB명 --col 컬렉션명 --filter '{"keyword":"테스트"}'

# 다중 삭제 (--many)
cd ~/Documents/claude_skills/mongo_manager && python3 mongo_manager.py delete --db DB명 --col 컬렉션명 --filter '{"duration":"SHORTS"}' --many
```

### 컬렉션 통계
```bash
cd ~/Documents/claude_skills/mongo_manager && python3 mongo_manager.py stats --db DB명 --col 컬렉션명
```

### 시트로 저장 (export-sheet)
```bash
# 기본 (디폴트 폴더에 새 시트 생성)
cd ~/Documents/claude_skills/mongo_manager && python3 mongo_manager.py export-sheet --db DB명 --col 컬렉션명

# 필터/정렬/limit 포함
cd ~/Documents/claude_skills/mongo_manager && python3 mongo_manager.py export-sheet --db DB명 --col 컬렉션명 --filter '{"duration":"SHORTS"}' --sort -views --limit 100 --title "쇼츠_조회수순"

# 기존 시트에 탭 추가
cd ~/Documents/claude_skills/mongo_manager && python3 mongo_manager.py export-sheet --db DB명 --col 컬렉션명 --sheet-id 시트ID --tab 탭이름

# 다른 폴더에 저장
cd ~/Documents/claude_skills/mongo_manager && python3 mongo_manager.py export-sheet --db DB명 --col 컬렉션명 --folder-id 폴더ID
```

디폴트 저장 폴더: `1386T_3BfE5XpD0a2EHKf_Vvm9kYmcitj`

### TTL 인덱스 (자동 삭제)

```bash
# TTL 인덱스 생성 (날짜 필드 기준 N초 후 자동 삭제)
cd ~/Documents/claude_skills/mongo_manager && python3 mongo_manager.py ttl-create --db DB명 --col 컬렉션명 --field createdAt --seconds 604800

# 자주 쓰는 seconds 값
# 3600   = 1시간
# 86400  = 1일
# 604800 = 7일
# 2592000 = 30일

# 인덱스 목록 조회 (TTL 여부 표시)
cd ~/Documents/claude_skills/mongo_manager && python3 mongo_manager.py ttl-list --db DB명 --col 컬렉션명

# TTL 인덱스 삭제
cd ~/Documents/claude_skills/mongo_manager && python3 mongo_manager.py ttl-drop --db DB명 --col 컬렉션명 --name 인덱스명
```

---

## 사용자 요청 처리

- "DB 목록 보여줘" → list-dbs 실행
- "03_project_ytb_gotrap 컬렉션 목록" → list-cols --db 03_project_ytb_gotrap
- "gotrap_keywords_비빔면 더블루 읽어줘" → read --db ... --col ... --limit 20
- "최근 10개" → --sort -_id --limit 10
- "keyword에서 라면 검색" → search --field keyword --query 라면
- "이 도큐 추가해줘" → insert --data '...'
- "views 필드를 999로 바꿔줘" → update --filter ... --set ...
- "이 도큐 삭제해줘" → delete --id 또는 --filter
- "컬렉션 통계" → stats
- "TTL 인덱스 만들어줘" → ttl-create (필드명, 기간 확인 후 실행)
- "인덱스 목록 보여줘" → ttl-list
- "TTL 인덱스 삭제해줘" → ttl-drop
- "시트로 저장해줘" / "시트에 저장해줘" → export-sheet (디폴트 폴더에 새 시트 생성)
- "이 시트에 저장해줘" + 시트 URL 전달 → --sheet-id 추출해서 탭 추가
- "이 폴더에 저장해줘" + 폴더 URL 전달 → --folder-id 추출해서 저장
- read/search 조건(filter, sort, limit)을 그대로 export-sheet에도 적용 가능

URL 형식으로 받으면 DB명/컬렉션명을 파싱해서 사용해.
예: /explorer/67b5.../03_project_ytb_gotrap/gotrap_keywords_비빔면 더블루/find
→ db=03_project_ytb_gotrap, col=gotrap_keywords_비빔면 더블루

결과는 한국어로 정리해서 보여줘.
