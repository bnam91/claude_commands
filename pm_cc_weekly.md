pm-cc(커머스 프로젝트)가 이번주 weekly meeting 토글에 PM 브리핑을 자동 작성하는 스킬이야.

## 실행

```bash
node ~/Documents/claude_skills/agent_meeting/agent_pm_input.js --pm pm-cc --type weekly
```

## 동작 방식

1. 이번주 weekly meeting 페이지 자동 탐색 (없으면 오류)
2. pm-cc 프로젝트 DB에서 현황 자동 분석
   - 업무막힘 → 🚨 블로킹 이슈
   - 진행 중 → 🔄 이번주 집중 목표
   - 진행대기(상위 3건) → 📌 다음 착수 예정
   - 막힘 + 마감임박 7일 이내 → 👤 현빈 지원 요청
3. 상황에 맞는 💬 PM 코멘트 자동 생성
4. "pm-cc — 이번주 목표" 토글에 구조화된 섹션으로 덮어쓰기

## 직접 내용 입력 (자동조회 대신)

```bash
node ~/Documents/claude_skills/agent_meeting/agent_pm_input.js \
  --pm pm-cc --type weekly \
  --content "목표1\n목표2\n현빈 지원요청: ..."
```

## 출력 구조 (노션 토글 내부)

```
🚨 블로킹 이슈
  • 업무명 [우선순위] — 진행 불가 상태

🔄 이번주 집중 목표
  • 업무명 [우선순위] (마감 YYYY-MM-DD)

📌 다음 착수 예정
  • 업무명 [우선순위]

👤 현빈 지원 요청
  • 업무명 — 의사결정/외부연락 필요

💬 PM 코멘트
  [상황 종합 판단 한 줄]
```

## 사용자 요청 처리

- "pm-cc 주간 브리핑 입력해줘" / "pm-cc 이번주 목표 넣어줘" → 자동 실행
- 직접 내용 지정 시 → `--content` 옵션 사용
