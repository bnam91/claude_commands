별도의 터미널 창에서 새로운 Claude Code 세션을 실행하고 프롬프트를 전달하는 스킬이야.

## 중요
사용자가 프롬프트 없이 호출하면 반드시 "전달할 프롬프트를 입력해주세요." 라고 물어봐.
프롬프트가 있을 때만 아래를 실행해.

## 실행 방법

사용자가 전달한 프롬프트를 PROMPT 자리에 넣어서 Bash로 실행해줘:

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
    do script "PROMPT" in w
    delay 0.5
    tell application "Terminal" to activate
    tell application "System Events"
        keystroke return
    end tell
end tell
EOF
```

실행 후 "Claude Jr.에게 프롬프트를 전달했습니다: [프롬프트 내용]" 라고 알려줘.
