네이버 메일 자동 발송을 도와주는 스킬이야.
스킬 경로: ~/Documents/github_skills/naver_mail

## 메일 보내는 법 알려달라고 하면

아래 두 가지 내용을 읽어서 보여줘:

1. note.txt 파일 읽기:
```bash
cat ~/Documents/github_skills/naver_mail/note.txt
```

2. *video_logs 폴더 안의 파일 목록 확인:
```bash
ls ~/Documents/github_skills/naver_mail/*video_logs/
```

파일 목록과 note.txt 내용을 한국어로 정리해서 보여줘.

## 입력 시트 안내

**메일 본문 템플릿 시트:**
https://docs.google.com/spreadsheets/d/1x7cj51TufSmJq-2iJjKdMIL8y-SnD0yJH-0KQxS-X74/edit?gid=0#gid=0

**메일 주소 시트:**
https://docs.google.com/spreadsheets/d/1yG0Z5xPcGwQs2NRmqZifz0LYTwdkaBwcihheA13ynos/edit?gid=1906694512#gid=1906694512

메일 발송 전 준비해야 할 것:
- 메일 주소 시트에 받는 사람 주소 입력
- 메일 본문 시트에 템플릿 준비

## 메일 발송 실행 (Claude Code 전용 흐름)

사용자가 메일을 보내려고 하면 아래 순서대로 진행해줘:

### Step 1: 옵션 목록 가져오기
```bash
cd ~/Documents/github_skills/naver_mail && node claude_runner.js --list
```
템플릿 시트 목록, DB 목록, 네이버 프로필 목록이 출력됨.

### Step 2: 사용자에게 선택 물어보기
각 목록을 보여주고 사용자에게 번호를 선택하도록 물어봐줘:
- 어떤 템플릿 시트를 쓸지 (번호)
- 어떤 DB(메일주소)를 쓸지 (번호)
- 어떤 네이버 프로필을 쓸지 (번호)

### Step 3: 선택값으로 터미널 열어서 발송 실행
사용자가 번호를 입력하면 새 터미널 창을 열어서 실행해줘 (번호는 사용자 선택값으로 교체):
```bash
osascript -e 'tell application "Terminal"
  activate
  do script "cd ~/Documents/github_skills/naver_mail && node claude_runner.js --template 1 --db 1 --profile 1"
end tell'
```
터미널에서 실행되므로 사용자가 Ctrl+C로 언제든 중단 가능해.

## 사용자 요청별 처리

- "메일 보내는 법 알려줘" / "사용법" → note.txt + *video_logs 파일 목록 보여주기
- "메일 보내줘" / "발송 시작" → Step 1~3 순서대로 진행
- 입력 시트 URL 안내 요청 → 위의 두 시트 URL 알려주기

결과는 한국어로 정리해서 보여줘.
