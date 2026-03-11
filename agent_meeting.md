rich-agent ↔ PM 에이전트 간 회의를 Notion meeting 테이블로 관리하는 스킬이야.
스크립트 경로: ~/Documents/claude_skills/agent_meeting/agent_meeting.js
Notion DB: 31c111a5778880a68164f8f27f2463c8 (meeting)

## 참여 PM 목록
- pm-cc, pm-gg, pm-xx

## 주요 명령

### 오전 회의 생성
```bash
node ~/Documents/claude_skills/agent_meeting/agent_meeting.js --morning
node ~/Documents/claude_skills/agent_meeting/agent_meeting.js --morning --date 2026-03-09
```

### 회의 목록 조회
```bash
node ~/Documents/claude_skills/agent_meeting/agent_meeting.js --list
node ~/Documents/claude_skills/agent_meeting/agent_meeting.js --list --date 2026-03-08
```

### 회의 페이지 읽기 (PM 입력 내용 확인)
```bash
node ~/Documents/claude_skills/agent_meeting/agent_meeting.js --read --id <page_id>
```

### 토글에 내용 쓰기
```bash
# Rich Agent 피드백 입력
node ~/Documents/claude_skills/agent_meeting/agent_meeting.js \
  --write --id <page_id> --block "Rich Agent 피드백" --text "피드백 내용"

# 최종 결과 입력
node ~/Documents/claude_skills/agent_meeting/agent_meeting.js \
  --write --id <page_id> --block "오전미팅 결과" --text "최종 스케줄"
```

## 오전 회의 페이지 구조
```
📢 [콜아웃] PM 브리핑 요청
  ▶ pm-cc         ← PM이 브리핑 + 업무요청 입력
  ▶ pm-gg
  ▶ pm-xx
🤖 [콜아웃] Rich Agent 피드백 안내
  ▶ Rich Agent 피드백   ← rich-agent가 종합 피드백 입력
  ▶ pm-cc (피드백 반영) ← PM이 재요청 반영
  ▶ pm-gg (피드백 반영)
  ▶ pm-xx (피드백 반영)
✅ [콜아웃] 최종 스케줄링
  오전미팅 결과 및 내용 :   ← 최종 확정 스케줄
```

## 사용자 요청 처리
- "오전 회의 열어줘" → `--morning` 실행
- "오늘 회의 목록 보여줘" → `--list --date 오늘날짜` 실행
- "회의 내용 읽어줘" → `--read --id <id>` 실행
- "pm-cc 토글에 써줘" → `--write --block "pm-cc" --text "..."` 실행

결과는 한국어로 정리해서 보여줘.
