---
name: port-status
description: 현재 CDP/디버깅 포트 연결 상태를 확인하고 유의사항을 안내하는 스킬. 사용자가 "포트 확인해줘", "포트 상태", "어떤 포트 열려있어", "CDP 포트 확인" 등을 말할 때 실행해.
---

# CDP 포트 상태 확인

## 실행 순서

### 1단계: 실행 중인 CDP 프로세스 확인

```bash
ps aux | grep "remote-debugging-port" | grep -v grep
```

### 2단계: 각 포트 응답 확인

```bash
for PORT in 9222 9333 9334; do
  RESULT=$(curl -s --connect-timeout 1 http://localhost:$PORT/json/version 2>/dev/null)
  if [ -n "$RESULT" ]; then
    BROWSER=$(echo $RESULT | python3 -c "import json,sys; d=json.load(sys.stdin); print(d.get('Browser','?'))" 2>/dev/null)
    echo "✅ $PORT 열림 — $BROWSER"
  else
    echo "❌ $PORT 닫힘"
  fi
done
```

### 3단계: MCP 연결 상태 확인

`mcp__chrome-devtools__list_pages()` 로 현재 MCP가 어느 포트에 붙어있는지 확인.

---

## 포트 정의표

| 포트 | 대상 | MCP | 조작 가능 여부 |
|------|------|-----|----------------|
| **9334** | Electron 앱 렌더러 (sangpe-editor) | `chrome-devtools` | ⚠️ 자동화 한정, kill 절대 금지 |
| **9333** | Electron 앱 서브 렌더러 | — | ⚠️ kill 절대 금지 |
| **9222** | 일반 Chrome (coq3820 프로필) | `chrome-devtools-9222` | ✅ 자유롭게 제어 가능 |

---

## ⚠️ 유의사항

1. **9333, 9334 kill 금지** — `npm run dev`로 실행된 Electron 앱. 종료 시 작업 중인 에디터 세션 날아감
2. **MCP `chrome-devtools`는 9334(일렉트론) 고정** — 이 MCP로 navigate/click 하면 Electron 앱이 조작됨. 의도치 않은 페이지 이동 주의
3. **일반 Chrome 작업은 `chrome-devtools-9222` 사용** — 9222가 닫혀있으면 `/chrome-cdp -coq3820` 으로 먼저 열어야 함
4. **9222가 닫혀있을 때** → `/chrome-cdp -coq3820` 실행해서 열기

---

## 빠른 참조

```bash
# 일반 Chrome 열기 (coq3820)
/chrome-cdp -coq3820

# Electron 앱 시작
cd /Users/a1/web-editor && npm run dev

# Electron DevTools 직접 열기 (브라우저에서)
open "http://localhost:9334/json"
```
