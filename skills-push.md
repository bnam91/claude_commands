Claude 스킬 관련 폴더들을 git add, commit, push 해줘.

대상 폴더:
- ~/.claude/commands
- ~/.claude/skills
- ~/Documents/claude_skills

## 처리 흐름

각 폴더에 대해 순서대로:
1. `git -C {폴더} status` 로 변경사항 확인
2. 변경사항 있으면 `git -C {폴더} add -A` 로 전체 스테이징
3. 변경된 파일 목록 바탕으로 한국어 커밋 메시지 작성 (예: "스킬 추가: scheduler, imgbb 업데이트")
4. `git -C {폴더} commit -m "커밋메시지"` 실행
5. `git -C {폴더} push` 실행
6. 변경사항 없으면 "변경사항 없음" 으로 스킵

## 커밋 메시지 규칙
- 한국어로 작성
- 추가된 파일: "스킬 추가: 파일명"
- 수정된 파일: "스킬 수정: 파일명"
- 삭제된 파일: "스킬 삭제: 파일명"
- 복합: "스킬 업데이트: 추가 X개, 수정 Y개"
- Co-Authored-By 라인 불필요

완료 후 세 폴더의 결과를 요약해서 보여줘.
