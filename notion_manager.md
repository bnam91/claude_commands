Notion 페이지/데이터베이스를 읽고, 쓰고, 수정하는 스킬이야.
스크립트 경로: ~/Documents/claude_skills/notion_manager/notion_manager.js

## 초기 설정

1. `config.example.json` → `config.json` 으로 복사
2. 아래 3가지 항목 입력:
   - `api_key`: Notion API 키 (https://www.notion.so/my-integrations 에서 발급)
   - `my_task_toggle_url`: 내 업무요청 토글 페이지 URL
   - `colleagues`: 동료들의 업무요청 토글 URL (이름 + URL)
3. 사용할 페이지/DB에 Integration 연결 필요

> URL은 Notion 페이지 열고 → 우측 상단 "공유" → "링크 복사"로 가져오면 돼.
> 설정 안 된 상태로 실행하면 자동으로 설정 안내가 출력돼.

## 사용 가능한 기능

### 페이지 조회
```bash
node ~/Documents/claude_skills/notion_manager/notion_manager.js get-page <page_id>
```

### 페이지 내용(블록) 읽기
```bash
node ~/Documents/claude_skills/notion_manager/notion_manager.js get-content <page_id>
```

### 데이터베이스 조회
```bash
node ~/Documents/claude_skills/notion_manager/notion_manager.js query-db <database_id>
```

### 페이지에 텍스트 추가
```bash
node ~/Documents/claude_skills/notion_manager/notion_manager.js append-text <page_id> "추가할 내용"
```

### 검색
```bash
# 전체 검색
node ~/Documents/claude_skills/notion_manager/notion_manager.js search "검색어"

# 타입 필터 (page 또는 database)
node ~/Documents/claude_skills/notion_manager/notion_manager.js search "검색어" page
```

### 페이지 삭제 (아카이브)
```bash
node ~/Documents/claude_skills/notion_manager/notion_manager.js delete-page <page_id>
```

## 코드에서 직접 사용 (import)

```javascript
import { getPage, queryDatabase, appendText, createPage } from './notion_manager.js';

// DB 조회
const result = await queryDatabase('database_id');

// 페이지에 텍스트 추가
await appendText('page_id', '내용');

// DB에 페이지 추가
await createPage('database_id', 'database', {
  이름: { title: [{ text: { content: '제목' } }] }
});
```

## page_id / database_id 찾는 법

Notion URL에서 추출:
`https://www.notion.so/페이지명-<여기32자리가_ID>`

하이픈 없이 붙여쓰거나 있어도 동작함.

결과는 한국어로 정리해서 보여줘.
