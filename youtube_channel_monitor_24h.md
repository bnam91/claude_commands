레퍼 채널들 중 최근 24시간 동안 업로드된 영상(Shorts 제외) 목록을 조회하는 스킬이야.

## 실행 방법

아래 명령을 Bash 도구로 실행해서 결과를 캡처해줘:

```bash
cd ~/Documents/github_skills/go-finder && node scripts/indicator_day.js
```

## 텔레그램 전송 (인자가 있을 때)

`$ARGUMENTS`가 있으면 아래 정보를 참고해서 해당 사용자에게 텔레그램으로도 전송해줘.

- 봇 토큰: `7849782487:AAEp6gwgun05PAH3Q7VSFbZ4D9-f4gga_qo` (finder 봇)
- 사용자 목록:
  - 지수: `8406936211`
  - 현빈: `6942656480`

인자에서 사용자 이름을 파악한 후, 아래 명령으로 전송해줘 (TEXT 부분에 스크립트 출력 결과 삽입):

```bash
curl -s -X POST "https://api.telegram.org/bot7849782487:AAEp6gwgun05PAH3Q7VSFbZ4D9-f4gga_qo/sendMessage" \
  -H "Content-Type: application/json" \
  -d "{\"chat_id\": \"<chatId>\", \"text\": \"<결과텍스트>\"}"
```

텔레그램 메시지 길이 제한(4096자)을 초과하면 여러 번 나눠서 전송해줘.

## 출력 처리

- 실행 결과를 한국어로 정리해서 보여줘.
- 채널별 구독자 수와 업로드 영상 목록을 보기 좋게 포맷해줘.
- 텔레그램 전송 시 성공/실패 여부를 알려줘.
- 오류 발생 시 오류 내용을 사용자에게 알려줘.
