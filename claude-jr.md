별도의 터미널 창에서 새로운 Claude Code 세션을 실행하고 프롬프트를 전달하는 스킬이야.

## 중요
사용자가 프롬프트 없이 호출하면 반드시 "전달할 프롬프트를 입력해주세요." 라고 물어봐.
프롬프트가 있을 때만 아래를 실행해.

## 실행 방법

특수문자·한글·줄바꿈이 포함될 수 있으므로 항상 클립보드 경유 방식을 사용해.

### Step 1: 프롬프트를 파일에 저장 후 클립보드에 복사
```bash
cat > /tmp/jr_prompt.txt << 'PROMPT'
[사용자 프롬프트 그대로]
PROMPT
pbcopy < /tmp/jr_prompt.txt
```

### Step 2: Claude Jr. 실행 + 클립보드 붙여넣기
```bash
osascript << 'EOF'
tell application "Terminal"
    activate
    set w to do script "claude"
    delay 5
    activate
    tell application "System Events"
        keystroke return
    end tell
    delay 5
    tell application "Terminal" to activate
    tell application "System Events"
        keystroke "v" using command down
        delay 0.5
        keystroke return
    end tell
end tell
EOF
```

실행 후 "Claude Jr.에게 프롬프트를 전달했습니다: [프롬프트 내용 요약]" 라고 알려줘.
