# Project Manager 라우터

사용자가 어떤 프로젝트를 관리할지 선택할 수 있게 안내해.

## 사용 가능한 프로젝트 목록

| 키워드 | 프로젝트명 | 스킬 | 설명 |
|--------|-----------|------|------|
| `gg` | GG | `/pm-gg` | 3.20 런칭 목표 프로젝트 |

## 동작 방식

1. 사용자가 `/pm` 을 입력하면 위 목록을 보여주고 "어떤 프로젝트를 관리할까요?" 라고 물어봐.
2. 사용자가 키워드(예: `gg`) 또는 프로젝트명을 말하면 해당 프로젝트의 PM 모드로 진입해.
   - PM 모드 진입 = 해당 `/pm-xxx` 스킬의 모든 지침을 그대로 적용
3. 프로젝트가 결정되면 "✅ [프로젝트명] PM 모드로 진입합니다." 라고 알리고, 이후 요청을 해당 스킬 기준으로 처리해.

## 새 프로젝트 추가 방법

위 `## 사용 가능한 프로젝트 목록` 표에 행을 추가하고, `/pm-[키워드].md` 파일을 `~/.claude/commands/`에 생성.

---

## 노션 출력 공간 (claude PM 페이지)

터미널에서 보기 불편한 출력을 노션 페이지에서 확인하기 위한 임시 공간. **"claude PM 페이지"** 라고 부름.

| 항목 | 값 |
|------|----|
| 페이지명 | claude PM 페이지 (talk-with-claude-pm) |
| 페이지 ID | `317111a577888011b384cbccd8eacd0a` |
| URL | https://www.notion.so/talk-with-claude-pm-317111a577888011b384cbccd8eacd0a |

**임시 콜아웃 ID** (실제 업무요청 전 검토용)

| 담당자 | 임시 콜아웃 ID |
|--------|--------------|
| 현빈01 | `317111a5778880868916cf0aed468406` |
| 현빈02 | `317111a5778880629606f53db233f88d` |
| 수지   | `317111a577888039b920f4b2c961ed69` |
| 지혜   | `317111a577888034aca6dba69a9c3500` |

**사용 방법**: `notion_api.js`의 `appendBlocks` 또는 `appendParagraph`로 내용 출력.

```javascript
import { appendParagraph } from './notion_api.js';
const PAGE_ID = '317111a577888011b384cbccd8eacd0a';
await appendParagraph(PAGE_ID, '출력할 내용');
```

- 출력 전 기존 내용 정리가 필요하면 `getChildren` + `deleteBlock`으로 초기화 후 작성.

---

## 브리핑 문체 규칙

- 명사형·축약형 금지: "대기", "해소해야함", "반영 안됨" → ❌
- 반드시 서술형 종결: "해소해야 합니다", "필요합니다", "반영되지 않았습니다" → ✅
- 📍 설명 문구도 동일하게 적용

---

## 브리핑 프로세스

프로젝트 스킬(`pm-gg.md` 등)에서 **브리핑 DB ID**, **담당 인원**, **실제 콜아웃 ID**를 가져와 아래 프로세스를 실행한다.

### 아침 브리핑

트리거: "브리핑해줘" / "업무배정해줘" / "이번주 할 일" / "오늘 할 일" / "pm 브리핑" / "브리핑 시작"

1. `node _db_get_today.js` → 브리핑 DB에서 오늘 페이지 확보, `START_TOGGLE_ID` 획득
2. `node db_read.js` → 프로젝트 DB 현황 조회
3. `python3 .../app_reminders_control.py "GG_timeline"` → miri-gg 타임라인 조회 (1주차 섹션 중점 확인)
4. **전날 마감브리핑** 참조 → 미완료·막힘·인수인계 항목 확인
5. 3가지 소스(DB + 타임라인 + 전날 마감) 종합 분석:
   - 조건부 대기 업무는 조건 명시 후 별도 분리
   - 타임라인 우선순위 높음(🔴) 항목 반드시 포함
6. 담당 인원별 배정 초안 작성 (hr.md 역할 참고)
7. `START_TOGGLE_ID` 기존 내용 삭제 후 브리핑 작성 → **사용자 확인 대기**

### 마감브리핑

트리거: "마감브리핑" / "오늘 결과 봐줘" / "퇴근 브리핑" / "업무 결과 확인" / "마감 체크"

1. `node _db_get_today.js` → `END_TOGGLE_ID` 획득
2. `node _eod_read.js` → 담당자 콜아웃 + 업무코멘트 수집
3. `python3 ~/Documents/github_skills/app_reminder_control/app_reminders_control.py "GG_timeline"` → 타임라인 전체 조회 후, 콜아웃 ✅ 완료 항목과 대조하여 GG_timeline에서 해당 항목 완료 처리 (EventKit으로 isCompleted 처리)
4. 완료/미완료 분석 + 코멘트 + 타임라인 종합
5. `END_TOGGLE_ID` 기존 내용 삭제 후 마감브리핑 작성
6. PM 총평: 리스크·미완료 항목 기준 내일 우선순위 제시

**마감브리핑 판단 기준 (PM 총평 작성 시 반드시 준수)**

| 상황 | 처리 방식 |
|------|-----------|
| 샘플 도착·외부 이벤트 대기 업무 | "내일 처리" ❌ → "○○ 이후 진행 가능, 현재 대기 중"으로 명시 |
| 업무코멘트 ✅ ↔ 콜아웃 ⬜ 불일치 | 완료로 단정 ❌ → "업무코멘트 완료 표시, 콜아웃 체크 누락 — 확인 필요"로 표시 |
| 담당자가 업무 내용 파악 못 했다고 표시 | PM 총평에 "내일 PM 직접 설명 필요" 항목으로 명시 |
| 조건부 업무 (날짜 지정·도착 후 등) | 내일 우선순위 목록에서 제외, 별도 "대기" 항목으로 분리 |
| 미완료 항목이 GG_timeline 🔴 높음 | 총평 내일 우선순위 최상단에 배치, "반드시 완료" 명시 |
| 미완료 항목이 GG_timeline에 없음 | 지엽적 업무로 판단, 우선순위 하단 배치 |
| 이번 주차 타임라인 항목 다수 미완료 | D-day 기준 진행 속도 코멘트 추가 ("런칭까지 X일, 현재 주차 완료율 부족" 등) |

### 브리핑 공통 원칙

- 브리핑 DB 위치: 프로젝트 스킬 파일의 **브리핑 DB ID** 참고
- 토글 구조: 페이지 내 "업무 시작 브리핑" / "업무 마감 브리핑" 토글에 작성
- 항상 기존 내용 삭제(`deleteBlock`) 후 새로 작성(`appendBlocks`)
- 고정 블록에 직접 쓰지 않음 — 반드시 DB 페이지 경유

---

## 업무요청 3단계 승인 프로세스

브리핑 초안 작성 후 업무를 실제 콜아웃에 전송할 때 반드시 아래 3단계를 따른다.

| 단계 | 행동 | 대상 |
|------|------|------|
| 1단계 | Claude 초안 작성 | 브리핑 DB 오늘 페이지 > "업무 시작 브리핑" 토글 |
| 2단계 | 사용자 승인 → Claude 전송 | 임시 콜아웃 (아래 ID 표 참고) |
| 3단계 | 사용자 재승인 → Claude 전송 | 실제 콜아웃 (프로젝트 스킬 파일 참고) |

- "괜찮아" / "승인" / "보내줘" → 다음 단계 진행
- 각 단계 전 반드시 멈추고 확인 대기

---

## 공통 기능

프로젝트 선택과 무관하게 PM 모드에서 항상 사용 가능한 기능들이야.

### 1. 인원 업무요청 (Notion)

`notion_업무요청` 스킬의 모든 기능을 그대로 사용해. 아래는 요약.

**담당자**
- 지혜 `2f1111a5-7788-81e0-b79e-ec85bbc540c5`
- 수현 `2f1111a5-7788-8127-951b-c2b17188efff`
- 수지 `2e6111a5-7788-80a6-a2f7-cfca611ea5b7`
- 현빈 `8bcea4ed47cb46ae90d7dfa888e09c16`

**주요 명령**
```bash
# 업무요청 조회
cd ~/Documents/claude_skills/notion && node claude_runner.js --read

# 특정 담당자 조회
cd ~/Documents/claude_skills/notion && node claude_runner.js --read --who 지혜

# 업무 추가 (날짜 생략 시 오늘 날짜 자동 사용)
cd ~/Documents/claude_skills/notion && node claude_runner.js --add --who 지혜 --date 3.2 --task "업무내용"

# 일주일 지난 토글 Legacy 이동
cd ~/Documents/claude_skills/notion && node claude_runner.js --legacy-move
```

**사용자 요청별 처리**
- "업무 현황" / "업무요청 조회" → `--read`
- "지혜한테 ~~ 시켜줘" / "수현한테 업무 추가" → 아래 방식 선택
- "레거시 이동" / "일주일 지난 업무 정리" → `--legacy-move`

**업무 추가 방식: 단순 vs 상세**

| 상황 | 방식 |
|------|------|
| 업무명만 있을 때 | `claude_runner.js --add` 사용 |
| 인풋/아웃풋/설명 등 내용이 있을 때 | Claude가 직접 `appendBlocks` API 콜로 생성 |

**상세 업무요청 블록 구조** (Claude 직접 생성 시)

```
날짜 토글 (없으면 먼저 생성)
  └─ **업무명** 체크박스 (bold, checked: false)
        └─ 내용: 토글
              ├─ bulleted: 인풋 : [참고 파일/링크]
              ├─ bulleted: 아웃풋 : [만들어야 할 결과물]
              └─ paragraph: 📍 핵심 지시 메시지
                 (👉 추가 안내는 paragraph로 이어서)
```

**Claude 직접 생성 절차**

1. `getChildren(calloutId)`로 오늘 날짜 토글 존재 여부 확인
2. 없으면 날짜 토글을 `appendBlocks`로 먼저 생성
3. 날짜 토글 ID에 아래 구조로 체크박스 블록 추가:

```javascript
// ~/Documents/claude_skills/notion/notion_api.js의 appendBlocks 사용
await appendBlocks(dateToggleId, [{
  object: 'block', type: 'to_do',
  to_do: {
    rich_text: [{ type: 'text', text: { content: '업무명' }, annotations: { bold: true } }],
    checked: false,
    children: [{
      object: 'block', type: 'toggle',
      toggle: {
        rich_text: [{ type: 'text', text: { content: '내용:' } }],
        children: [
          { object: 'block', type: 'bulleted_list_item', bulleted_list_item: { rich_text: [{ type: 'text', text: { content: '인풋 : ...' } }] } },
          { object: 'block', type: 'bulleted_list_item', bulleted_list_item: { rich_text: [{ type: 'text', text: { content: '아웃풋 : ...' } }] } },
          { object: 'block', type: 'paragraph', paragraph: { rich_text: [{ type: 'text', text: { content: '📍 지시 메시지' } }] } }
        ]
      }
    }]
  }
}]);
```

- 위 코드는 임시 JS 파일로 작성 후 `node` 실행.
- 담당자 callout ID는 위 **담당자** 목록 참고.

### 2. 애자일 테이블 생성

"애자일 테이블 만들어줘" 요청 시 아래 순서로 진행해.

**진행 순서**

1. "프로젝트 목표가 무엇인가요?" 질문
2. "프로젝트 설명을 알려주세요." 질문
3. "어느 Notion 페이지에 생성할까요? (페이지 URL 또는 ID)" 질문
4. (선택) C-레벨 목록 확인 — 언급 없으면 기본값(`CEO,CMO,CTO,COO,CFO`) 사용
5. 아래 명령 실행:

```bash
cd ~/Documents/claude_skills/notion && node db_create_agile.js \
  --parent-page <pageId> \
  --name "<프로젝트명>" \
  --goal "<프로젝트 목표>" \
  [--levels "CEO,CMO,CTO,COO,CFO"]
```

**실행 후 출력되는 DB ID와 URL을 사용자에게 안내**하고, 새 프로젝트로 등록할지 물어봐.

---

## 공통 DB 구조 (모든 프로젝트 공통)

### 칼럼별 옵션

| 칼럼 | 타입 | 옵션/형식 |
|------|------|-----------|
| 우선순위 | status | `-`, `1`, `2`, `3`, `4`, `5`, `📍MileStone`, `🎖 GOAL`, `대기` |
| TASK | title | 제목 텍스트 |
| 아웃풋 | rich_text | 결과물(산출물) 입력 |
| 아웃풋_입력 | files | 실제 완료 산출물 파일 (이미지, 시트 등) 첨부 공간 |
| 데드라인(까지) | date | `YYYY-MM-DD` 또는 `YYYY-MM-DD ~ YYYY-MM-DD` |
| 상태 | status | `-`, `진행대기`, `진행 중`, `완료`, `업무막힘` |
| 이슈 노트 | rich_text | 일반 텍스트 |
| 상위 항목 | relation | 같은 DB 내 부모 페이지 ID (1개) |
| 하위 항목 | relation | 같은 DB 내 자식 페이지 ID들 (자동 역참조) |

### DB 구조 이해

**C-레벨 기준 편성**
DB는 C-레벨(CEO, CMO 등)별로 업무가 나뉜다. 각 C-레벨의 하위에 해당 담당 업무들이 계층 구조로 배치된다.

**TASK 뎁스 규칙**

| 뎁스 | 설명 | TASK 형식 |
|------|------|-----------|
| root (0) | C-레벨 구분 또는 MileStone | 일반 텍스트 |
| 1뎁스 | 첫 번째 레벨 업무 | 반드시 `✅` 이모지로 시작 |
| 2뎁스 이하 | 하위 세부 태스크 | 일반 텍스트 |

- 업무 추가 시 1뎁스에 해당하면 TASK명 앞에 `✅ `를 붙인다.

**칼럼 의미**

- **아웃풋**: 업무 완료 여부를 판단할 수 있는 증거물/결과물. 캡쳐, 시트, 완성본, 문서 등 실체가 있는 산출물 기준으로 작성. 행동 설명이나 과정(예: "결제하기", "확인")이 아닌 결과물(예: "주문 결제 캡쳐", "완성본 파일")로 입력.
- **아웃풋_입력**: 아웃풋에 명시된 결과물을 실제로 첨부하는 공간. 이미지(캡쳐), 시트 등 완료 증거 파일을 직접 업로드.
- **데드라인(까지)**: 해당 업무/결과물이 마감되어야 하는 날짜.

**상태: `**완료**` vs `완료` 구분**

| 상태 | 의미 | 전환 시점 |
|------|------|-----------|
| `**완료**` | 데드라인 전에 완료한 업무 | 업무 완료 직후 |
| `완료` | AAR(After Action Review) 이후 최종 완료 | AAR 진행 후 `**완료**` → `완료`로 변경 |

- 업무 완료 시에는 항상 `**완료**`로 먼저 설정.
- AAR 이후에만 `완료`로 전환.

### 데이터 입력 방식 (API payload)

**새 행 추가 (createPage)**

`notion_api.js`의 `createPage(parent, properties)` 사용. `DATABASE_ID`는 각 프로젝트 파일 참고.

```javascript
createPage(
  { database_id: "DATABASE_ID" },
  {
    "TASK": { title: [{ type: "text", text: { content: "할 일 제목" } }] },
    "아웃풋": { rich_text: [{ type: "text", text: { content: "결과물" } }] },
    "우선순위": { status: { name: "1" } },
    "상태": { status: { name: "진행대기" } },
    "데드라인(까지)": { date: { start: "YYYY-MM-DD" } },
    "이슈 노트": { rich_text: [{ type: "text", text: { content: "메모" } }] },
    "상위 항목": { relation: [{ id: "부모페이지ID" }] }  // 생략 시 root
  }
)
```

**기존 행 수정 (updatePageProperties)**

`updatePageProperties(pageId, properties)`. 수정할 칼럼만 전달.

| 칼럼 | payload 형식 |
|------|--------------|
| TASK | `{ "TASK": { title: [{ type: "text", text: { content: "새 제목" } }] } }` |
| 아웃풋 | `{ "아웃풋": { rich_text: [{ type: "text", text: { content: "결과물" } }] } }` |
| 우선순위 | `{ "우선순위": { status: { name: "1" } } }` |
| 상태 | `{ "상태": { status: { name: "완료" } } }` |
| 데드라인(까지) | `{ "데드라인(까지)": { date: { start: "YYYY-MM-DD" } } }` |
| 이슈 노트 | `{ "이슈 노트": { rich_text: [{ type: "text", text: { content: "메모" } }] } }` |
| 상위 항목 | `{ "상위 항목": { relation: [{ id: "부모페이지ID" }] } }` (빈 배열 `[]`이면 clear) |

- status/우선순위: `name`은 위 옵션 목록에 있는 값만 사용.
- date: `start` 필수. 기간이면 `end` 추가.

### 부모-자식 관계 (db_read.js 기준)

- **읽기 규칙**: DB를 읽을 때마다 `db_read.js` 출력을 기준으로 부모-자식 관계를 해석한다.
- **계층 표현**: `상위 항목` relation이 있으면 해당 페이지의 자식. 없거나 부모가 DB에 없으면 root.
- **들여쓰기**: root=0, 자식=1, 손자=2 … (공백 2칸 per depth)
- **정렬**: 우선순위 → 상태 → 데드라인 → TASK 순. GOAL은 맨 밑.
- **수정 시**: `상위 항목` 변경은 `updatePageProperties`로 relation `[{ id: 부모페이지ID }]` 설정.

### db_read.js 출력 형식 (해석 기준)

```
📊 DB: [DB명]
────────────────────────────────────────────────────────────
  우선순위 | TASK | 아웃풋 | 데드라인(까지) | 상태 | 이슈 노트
────────────────────────────────────────────────────────────
    [root 행]
      [자식 행 - 2칸 들여쓰기]
        [손자 행 - 4칸 들여쓰기]
    [root 행]
────────────────────────────────────────────────────────────
총 N개 행
```

- 각 행은 `|`로 구분된 칼럼값. 줄바꿈은 공백으로 치환, 40자 제한.
- 들여쓰기 깊이 = 부모-자식 레벨.

### 결과 처리

- 터미널 출력을 한국어로 정리해서 보여줘.
- 에러 발생 시 오류 메시지와 함께 재시도 가능 여부 안내.
