별도의 터미널 창에서 새로운 Claude Code 세션을 여는 스킬이야.

## 프롬프트 추출

ARGUMENTS에서 대괄호 `[...]` 안의 텍스트를 프롬프트로 사용해.
- 예: `[관련 스킬 푸시 해줘]` → 프롬프트: `관련 스킬 푸시 해줘`
- 대괄호가 없으면 실행하지 말고 "프롬프트를 대괄호로 전달해줘. 예: `/claude-jr-window [할 일]`" 라고 안내해줘.

## 실행 방법

먼저 OS를 감지해:
```bash
uname -s
```

### Windows인 경우 (MINGW, MSYS, CYGWIN 등)

**Step 1**: Write 툴로 `/tmp/claude_jr_prompt.txt` 파일에 추출한 프롬프트 텍스트만 저장해줘. (파일 내용 = 프롬프트 텍스트 그대로)

**Step 2**: 아래 명령을 Bash로 실행해줘 (`/tmp/claude_jr_window.ps1`은 고정 파일이야, 매번 안 써도 돼):

```bash
powershell.exe -ExecutionPolicy Bypass -WindowStyle Hidden -File "C:/tmp/claude_jr_window.ps1" -PromptFile "C:/tmp/claude_jr_prompt.txt"
```

실행 후 "Claude 창을 열었습니다." 라고 알려줘.
