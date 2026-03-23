GoAgent_Macbook_bot 텔레그램 채널을 tmux 백그라운드 세션으로 실행하는 스킬이야.
사용자가 "텔레그램 봇 켜줘", "goagent 텔레그램 시작", "telegram 채널 실행" 등을 말할 때 실행해.

## 동작

### 1. 이미 실행 중인지 확인

```bash
tmux has-session -t goagent-telegram 2>/dev/null && echo "RUNNING" || echo "NOT_RUNNING"
```

- **RUNNING** → "이미 실행 중이에요. (`tmux attach -t goagent-telegram` 로 확인 가능)" 라고 알리고 종료.
- **NOT_RUNNING** → 다음 단계 진행.

### 2. tmux 세션 생성 및 실행

```bash
tmux new-session -d -s goagent-telegram 'claude --channels plugin:telegram@claude-plugins-official --dangerously-skip-permissions'
```

실행 후 1초 대기:

```bash
sleep 1 && tmux has-session -t goagent-telegram 2>/dev/null && echo "OK" || echo "FAIL"
```

- **OK** → "GoAgent 텔레그램 봇이 백그라운드에서 시작됐어요. @GoAgent_Macbook_bot 으로 메시지 보내보세요." 라고 알려줘.
- **FAIL** → "세션 시작 실패. tmux가 설치되어 있는지 확인해주세요." 라고 알려줘.

### 3. 중지 요청 시 (`stop` 인자가 있을 때)

사용자가 `/goagent-telegram stop` 처럼 `stop` 인자를 전달한 경우:

```bash
tmux kill-session -t goagent-telegram 2>/dev/null && echo "STOPPED" || echo "NOT_FOUND"
```

- **STOPPED** → "텔레그램 봇 세션을 종료했어요."
- **NOT_FOUND** → "실행 중인 세션이 없어요."

## 참고

- 세션 이름: `goagent-telegram`
- 봇 이름: `@GoAgent_Macbook_bot`
- 토큰 위치: `~/.claude/channels/telegram/.env`
- 로그 확인: `tmux attach -t goagent-telegram` (나올 때: `Ctrl+B` → `D`)
