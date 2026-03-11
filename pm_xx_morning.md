pm-xx(역전야매치트키 유튜브 채널, Phase 1 진행 중)가 오늘 morning meeting 토글에 일일 브리핑을 입력하는 스킬이야.

## 프로젝트 컨텍스트

| 항목 | 내용 |
|------|------|
| 프로젝트명 | XX (역전야매치트키) |
| 현재 Phase | Phase 1 — 채널 가동 + 영상 10편 업로드 |
| 업로드 주기 | 주 1편 |
| 콘텐츠 방향 | 수익화 강의 커리큘럼 기반 (얼굴 없는 채널) |

## 실행

```bash
node ~/Documents/claude_skills/agent_meeting/agent_pm_input.js --pm pm-xx --type morning
```

## 동작 방식

1. 오늘 morning meeting 페이지 자동 탐색
2. "pm-xx" 토글에 내용 덮어쓰기
3. 자동조회 미구현 → `--content`로 직접 작성 필요

## 직접 내용 입력

```bash
node ~/Documents/claude_skills/agent_meeting/agent_pm_input.js \
  --pm pm-xx --type morning \
  --content "오늘 진행: 스크립트 작성\n현재 영상 진행: 2편 편집 중\n현빈 요청: 촬영 시간 배정 필요"
```

## 브리핑 항목 가이드

- **오늘 진행 예정**: 콘텐츠 제작 단계 기준 오늘 할 일
- **현재 Phase 1 현황**: 10편 중 몇 편 완료/진행 중
- **블로킹**: 스크립트 지연, 편집자 미채용 등
- **현빈 요청**: 촬영, 주제 결정, 검토 등 현빈이 처리해야 할 것

## 사용자 요청 처리

- "pm-xx 오전 브리핑" / "오늘 pm-xx 브리핑 써줘" → 실행
