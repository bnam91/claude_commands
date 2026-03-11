이번주 각 PM들의 달성 목표를 취합하는 주간 회의 페이지를 Notion meeting DB에 생성하는 스킬이야.
스크립트: ~/Documents/claude_skills/agent_meeting/agent_meeting.js

## 실행
```bash
# 이번주 주간 회의 생성
node ~/Documents/claude_skills/agent_meeting/agent_meeting.js --weekly

# 특정 날짜 기준 주 생성
node ~/Documents/claude_skills/agent_meeting/agent_meeting.js --weekly --date 2026-03-09
```

## 페이지 구조
```
📢 [콜아웃] PM 목표 입력 요청
  ▶ pm-cc — 이번주 목표
  ▶ pm-gg — 이번주 목표
  ▶ pm-xx — 이번주 목표
🤖 [콜아웃] Rich Agent 종합 피드백 안내
  ▶ Rich Agent 종합 피드백
  ▶ pm-cc (피드백 반영)
  ▶ pm-gg (피드백 반영)
  ▶ pm-xx (피드백 반영)
✅ [콜아웃] 최종 실행 플랜 확정
  주간 미팅 결과 및 내용 :
```

## 노션 DB
- meeting DB: 31c111a5778880a68164f8f27f2463c8
- 날짜: 월요일~일요일 범위로 자동 설정
- 상태: 예정

## 사용자 요청 처리
- "주간 회의 열어줘" / "이번주 목표 취합해줘" / "weekly meeting 만들어줘" → `--weekly` 실행

결과는 한국어로 정리하고 Notion URL 출력해줘.
