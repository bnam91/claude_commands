노션 업무요청 조회/추가 및 페이지 읽기/쓰기를 REST API로 직접 처리하는 스킬이야.
스킬 경로: ~/Documents/claude_skills/notion

## 콜아웃 사용 전 확인

**업무 조회/추가 전에 반드시 확인:** 콜아웃에 오늘 날짜의 토글이 있는지 확인한다. 날짜는 `2.27`, `2/27`, `3.2` 등 M.D 또는 M/D 형식으로 입력되어 있다. **오늘 날짜 토글이 없으면 먼저 해당 날짜 토글을 생성한 후** 업무를 추가한다.

## 담당자별 콜아웃
- **지혜**: `2f1111a5-7788-81e0-b79e-ec85bbc540c5`
- **수현**: `2f1111a5-7788-8127-951b-c2b17188efff`
- **수지**: `2e6111a5-7788-80a6-a2f7-cfca611ea5b7`
- **현빈**: `8bcea4ed47cb46ae90d7dfa888e09c16`

## Legacy(이전) 페이지
- **지혜**: [이전](https://www.notion.so/2f6111a57788808eae5defbaa785763f)
- **수현**: [이전](https://www.notion.so/302111a57788802fb1e2d4c8cbd59df3)
- **수지**: [이전](https://www.notion.so/2e6111a57788807bad70f81ef26fc2f7)
- **현빈**: [이전](https://www.notion.so/1cf111a5778880eabe57d809143caf40)

## 업무요청 조회

```bash
# 전체 담당자 조회
cd ~/Documents/claude_skills/notion && node claude_runner.js --read

# 특정 담당자만 조회
cd ~/Documents/claude_skills/notion && node claude_runner.js --read --who 지혜

# 토글만 보기 (하위 데이터 생략)
cd ~/Documents/claude_skills/notion && node claude_runner.js --read --summary --who 지혜
```

## Legacy 이동 (5일 지난 토글 → 이전 페이지)

```bash
# 전체 담당자
cd ~/Documents/claude_skills/notion && node claude_runner.js --legacy-move

# 특정 담당자만
cd ~/Documents/claude_skills/notion && node claude_runner.js --legacy-move --who 지혜
```
- 오늘 날짜로부터 5일 이상 지난 날짜 토글(하위 to_do 포함)을 각 담당자의 '이전' 페이지로 이동
- 앵커블록이 아닌 블록은 **삭제가 아닌 이전 페이지로 이동**
- **앵커블록**: 정리 시 유지. `child_page`(이전, inbox), `divider`, `toggle`(inbox, 📌로 시작, 5일 이내 날짜), `heading`(5일 이내 날짜), `paragraph`(업무요청_ 포함, 맨 끝 빈 것)
- 맨 끝에 빈 paragraph 블록이 항상 있도록 유지

## 업무 추가

**단건 추가 (claude_runner.js)**
```bash
cd ~/Documents/claude_skills/notion && node claude_runner.js --add --who 지혜 --date 3.2 --task "업무내용"
```

**다건/배치 추가 (task_writer.js)** ← 업무요청 시 기본 사용
```bash
cd ~/Documents/claude_skills/notion && node task_writer.js --json '[
  {
    "who": "지혜",
    "date": "3.17",
    "tasks": [
      { "task": "단순 업무" },
      { "task": "복잡 업무명", "background": "배경", "input": "인풋", "output": "아웃풋" }
    ]
  },
  {
    "who": "수지",
    "tasks": [{ "task": "업무내용" }]
  }
]'
```
- `date` 생략 시 오늘 날짜 자동 사용
- 여러 담당자 동시 요청 가능
- `background`, `input`, `output`, `comment` 중 하나라도 있으면 **[토글] 내용:** 자동 생성
- 해당 날짜 토글이 있으면 그 안에 체크박스 추가, 없으면 토글 새로 만들고 추가
- **날짜 미지정 시 오늘 날짜 사용**: `--date` 생략 시 M.D 형식(예: 3.2, 2.27) 또는 M/D 형식(예: 2/27)으로 오늘 날짜를 넣어 실행. 오늘 날짜 토글이 없으면 자동 생성된다.
- `--background`, `--input`, `--output`, `--comment` 중 하나라도 있으면 체크박스 안에 **[토글] 내용:** 이 자동 생성되고 그 안에 내용이 들어감

## 업무요청 작성 템플릿

### 단순 업무 (한 줄)
```
- [ ] 업무내용 (참고사항 또는 기한)
```

### 복잡 업무 (인풋/아웃풋 있는 경우)
```
- [ ] **업무명**
    - 내용:
        📍 배경이나 맥락 — 왜 이 업무를 요청하는지, 뭘 해야 하는지 설명

        - 인풋: 어디 있는 파일/시트 보면 되는지 (드라이브 링크, 시트명 등)
        - 아웃풋: 어디에 어떻게 넣으면 되는지 (시트명, 열, 형태 등)

        👉 현빈 코멘트 — 모르면 물어봐도 된다거나, 이것저것 선택지 줄 때, 주의사항 있을 때 쓰는 부분
```

**작성 원칙**
- 인풋/아웃풋 명확히 구분해서 쓰기
- 📍는 배경/맥락/핵심 지시
- 👉는 현빈이 직접 붙이는 코멘트
- 다 끝나면 `[x]`로 체크

## 페이지 읽기/쓰기

```bash
# 페이지 블록 조회
cd ~/Documents/claude_skills/notion && node claude_runner.js --page-read --page <pageId>

# 페이지에 단락 추가
cd ~/Documents/claude_skills/notion && node claude_runner.js --write --page <pageId> --text "내용"
```

## 사용자 요청별 처리

- "업무 현황" / "업무요청 조회" → `--read` 실행
- "지혜한테 업무 추가" / "수현한테 ~~ 시켜줘" / "현빈한테 ~~ 시켜줘" → `--add` 실행 (날짜 없으면 오늘 날짜 사용)
- "일주일 지난 업무 이전으로 옮겨줘" / "legacy 이동" → `--legacy-move` 실행
- "노션 페이지 읽어줘" → `--page-read` 실행
- "노션에 써줘" → `--write` 실행

결과는 한국어로 정리해서 보여줘.
