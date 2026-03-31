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

## DB 항목별 체크리스트 + 완료율 자동화 패턴

"각 행(항목)마다 하위 체크리스트를 두고, 체크하면 완료율이 자동 계산되게 하고 싶다"는 요구에 사용하는 패턴.

### 구조 설계

```
부모 DB (예: 자동화 스텝)
  └─ 완료율 (rollup: percent_checked)
  └─ 체크리스트 (relation → 자동화스텝)

자식 DB (예: Wing 체크리스트)
  └─ 항목명 (title)
  └─ 완료 (checkbox)
  └─ 스텝 (relation, dual_property → 부모 DB)
```

### 1단계: 자식 DB 생성 + relation 연결

```javascript
// Notion API로 자식 DB 생성
const childDb = await fetch('https://api.notion.com/v1/databases', {
  method: 'POST',
  headers: { 'Authorization': 'Bearer ' + apiKey, 'Content-Type': 'application/json', 'Notion-Version': '2022-06-28' },
  body: JSON.stringify({
    parent: { type: 'page_id', page_id: PARENT_PAGE_ID },
    title: [{ text: { content: '체크리스트 DB명' } }],
    properties: {
      '항목명': { title: {} },
      '완료': { checkbox: {} },
      '스텝': {
        relation: {
          database_id: PARENT_DB_ID,
          type: 'dual_property',
          dual_property: {}
        }
      }
    }
  })
});
```

### 2단계: 부모 DB에 rollup 추가

```javascript
// 완료율 rollup 속성 추가
await fetch('https://api.notion.com/v1/databases/' + PARENT_DB_ID, {
  method: 'PATCH',
  headers: { 'Authorization': 'Bearer ' + apiKey, 'Content-Type': 'application/json', 'Notion-Version': '2022-06-28' },
  body: JSON.stringify({
    properties: {
      '완료율': {
        rollup: {
          relation_property_name: '체크리스트',   // 부모 DB의 relation 컬럼명
          rollup_property_name: '완료',            // 자식 DB의 checkbox 컬럼명
          function: 'percent_checked'
        }
      }
    }
  })
});
```

### 3단계: 자식 항목 일괄 생성 + relation 연결

```javascript
// 각 부모 항목마다 자식 체크리스트 생성
for (const step of steps) {
  for (const item of step.checklistItems) {
    await fetch('https://api.notion.com/v1/pages', {
      method: 'POST',
      headers: { ... },
      body: JSON.stringify({
        parent: { database_id: CHILD_DB_ID },
        properties: {
          '항목명': { title: [{ text: { content: item } }] },
          '스텝': { relation: [{ id: step.pageId }] }  // 부모와 연결
        }
      })
    });
  }
}
```

> ⚠️ dual_property relation은 기존 데이터를 소급 적용하지 않음.
> single_property → dual_property 전환 후 기존 항목들이 역방향 연결 안 되면,
> 각 항목을 clear → re-PATCH하여 강제 재등록 필요.

### 4단계: 각 부모 항목 페이지에 Linked View 삽입 (CDP 필요)

Notion API로는 Linked View를 페이지 본문에 삽입 불가. CDP(Chrome DevTools)로 처리:

```javascript
// 각 스텝 페이지에서 실행할 자동화 시퀀스 (mcp__chrome-devtools-9222 사용)
const stepName = '해당 스텝명';

// 1. 페이지 본문 contenteditable 클릭
const editable = document.querySelector('.notion-page-content [contenteditable]');
editable.click(); editable.focus();

// 2. /linked 슬래시 커맨드 입력 → "링크된 데이터 소스 보기" 선택 → Enter
// 3. 검색창에 자식 DB명 입력 → 선택
// 4. 팝업 Escape → 필터 버튼 클릭 → "스텝" 선택 → 해당 스텝명 선택
// 5. Escape → "모두에게 저장" 클릭
```

전체 CDP 자동화 순서 (페이지 이동 후 실행):
```javascript
async function insertLinkedView(stepName) {
  const delay = ms => new Promise(r => setTimeout(r, ms));
  await delay(2000); // 페이지 로드 대기

  // 1. 본문 포커스
  const editable = document.querySelector('.notion-page-content [contenteditable]');
  editable.click(); editable.focus();
  await delay(300);

  // 2~3: type_text("/linked") → 옵션 선택 → DB 검색·선택은 외부 type_text 툴로
  // 4. 필터 설정
  const fb = document.querySelectorAll('[aria-label*="필터"]');
  fb[0]?.click();
  await delay(500);

  const opts = document.querySelectorAll('[role="option"]');
  opts.forEach(o => { if (o.textContent.trim() === '스텝') o.click(); });
  await delay(600);

  document.querySelectorAll('[role="option"]').forEach(o => {
    if (o.textContent.includes(stepName)) o.click();
  });
  await delay(500);

  // 5. 저장
  document.querySelectorAll('span, div').forEach(el => {
    if (el.textContent.trim() === '모두에게 저장' && el.children.length === 0) {
      (el.closest('[role="button"]') || el.parentElement)?.click();
    }
  });
}
```

### 완성 후 동작

- 부모 DB에서 각 행 클릭 → 페이지 본문에 해당 행의 체크리스트만 표시
- 체크박스 체크 → 부모 DB 완료율(%) 실시간 업데이트

---

## 기본 대상 페이지

사용자가 **"노션에 추가해줘"** 라고 하면 별도 URL 언급 없어도 아래 페이지에 서브페이지를 생성해:

- **CHARIZARD 페이지**: `https://www.notion.so/CHARIZARD-2ca111a5778880c99435efef0cbf7707`
- page_id: `2ca111a5778880c99435efef0cbf7707`

```bash
# 서브페이지 생성 예시
node ~/Documents/claude_skills/notion_manager/notion_manager.js create-page 2ca111a5778880c99435efef0cbf7707 page "페이지 제목"
```
