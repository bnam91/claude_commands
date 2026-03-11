pm-gg(GG 딜롱·사입 프로젝트, D-day 3.20 런칭)가 이번주 weekly meeting 토글에 PM 브리핑을 작성하는 스킬이야.

## 프로젝트 컨텍스트

| 항목 | 내용 |
|------|------|
| 프로젝트명 | GG (딜롱·사입) |
| 런칭 목표 | 2026-03-20 |
| 담당 인원 | 현빈02, 지혜 |
| 핵심 업무 | 샘플 소싱, 발주, 입고, 쿠팡 상품 등록 |

## 실행

```bash
node ~/Documents/claude_skills/agent_meeting/agent_pm_input.js --pm pm-gg --type weekly
```

## 동작 방식

1. 이번주 weekly meeting 페이지 자동 탐색
2. "pm-gg — 이번주 목표" 토글에 내용 덮어쓰기
3. pm-gg DB 자동조회 미구현 → `--content`로 직접 작성 권장

## 직접 내용 입력 (권장)

```bash
node ~/Documents/claude_skills/agent_meeting/agent_pm_input.js \
  --pm pm-gg --type weekly \
  --content "이번주 목표: 입고 완료\n블로킹: 샘플 미도착\n현빈 지원요청: 발주 승인"
```

## 브리핑 작성 가이드

주간 브리핑 작성 시 아래 항목을 포함해야 해:
- **이번주 집중 목표**: 런칭 D-day 기준 이번주에 반드시 완료할 것
- **블로킹 이슈**: 진행 불가 상태인 것 (외부 대기, 의사결정 필요)
- **현빈 지원 요청**: 현빈이 직접 처리해야 할 것 (발주 승인, 외부 연락 등)
- **PM 코멘트**: 런칭 일정 대비 현재 속도 평가 한 줄

## 사용자 요청 처리

- "pm-gg 주간 브리핑 입력해줘" / "pm-gg 이번주 목표 써줘" → 실행
