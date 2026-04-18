---
name: port-status
description: 현재 CDP/디버깅 포트 연결 상태를 확인하고 유의사항을 안내하는 스킬. 사용자가 "포트 확인해줘", "포트 상태", "어떤 포트 열려있어", "CDP 포트 확인" 등을 말할 때 실행해.
---

# CDP 포트 상태 확인

## 실행 순서

### 0단계: chrome-devtools-mcp 좀비 프로세스 확인 (항상 먼저 실행)

```bash
COUNT=$(ps aux | grep "chrome-devtools-mcp" | grep -v grep | wc -l | tr -d ' ')
echo "chrome-devtools-mcp 프로세스 수: ${COUNT}개"
if [ "$COUNT" -ge 20 ]; then
  echo "⚠️ 좀비 프로세스 누적 감지 — 정리를 권장합니다"
else
  echo "✅ 정상 범위"
fi
```

20개 이상이면 사용자에게 알리고 정리 여부를 물어본다. 정리 방법은 아래 4단계 참조.

---

### 1단계: 실행 중인 CDP 프로세스 확인

```bash
ps aux | grep "remote-debugging-port" | grep -v grep
```

### 2단계: 각 포트 응답 확인

```bash
for PORT in 9222 9333 9334 9335 9336 9337 9338 9339 9340 9341; do
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

### 4단계: chrome-devtools-mcp 좀비 프로세스 확인 및 정리

Claude Code 세션을 열 때마다 `chrome-devtools-mcp` 프로세스가 생성되는데, 세션이 닫혀도 프로세스가 남아 누적됨. 시스템이 느려지는 주요 원인.

```bash
# 누적된 MCP 프로세스 수 확인
ps aux | grep "chrome-devtools-mcp" | grep -v grep | wc -l
```

**20개 이상이면 좀비 정리 권장.** 단, 현재 세션 프로세스(가장 최근 PID 그룹)는 남겨야 함:

```bash
# 현재 세션 PID 확인 (가장 최근 시작된 npm exec 프로세스)
ps aux | grep "npm exec chrome-devtools-mcp" | grep -v grep | sort -k2 -n | tail -2

# 위에서 확인한 PID를 KEEP_ABOVE에 입력 후 실행
KEEP_ABOVE=<현재세션_최소PID>
ps aux | grep "chrome-devtools-mcp" | grep -v grep | awk -v k=$KEEP_ABOVE '$2 < k {print $2}' | xargs kill 2>/dev/null
echo "정리 완료. 남은 프로세스: $(ps aux | grep chrome-devtools-mcp | grep -v grep | wc -l)개"
```

> ⚠️ 현재 세션 MCP를 kill하면 도구가 끊기고 CC 재시작이 필요함. 정리 후 재시작하는 것을 권장.

**빠른 전체 정리 (CC 재시작 감수할 때):**
```bash
pkill -f "chrome-devtools-mcp" && echo "전체 정리 완료 — CC 재시작 필요"
```

---

## 포트 정의표

| 포트 | 대상 | MCP | 조작 가능 여부 |
|------|------|-----|----------------|
| **9333** | web-editor 서브 렌더러 (sangpe-editor) | — | ⚠️ kill 절대 금지 |
| **9334** | web-editor 메인 (`npm run dev`) | `chrome-devtools` | ⚠️ 자동화 한정, kill 절대 금지 |
| **9335** | web-editor `dev:step2` / go-finder | — | ⚠️ kill 금지 |
| **9336** | web-editor `dev:planning` (goditor admin) | — | ✅ goditor-layout-* 스킬 전용 |
| **9337** | web-editor `dev:ui-polish` (goditor 피그마) | — | ⚠️ goditor-figma 전용 — 다른 세션 사용 금지 |
| **9338** | web-editor `dev:design-token` | — | ⚠️ kill 금지 |
| **9339** | web-editor `dev:template-tag` | — | ⚠️ kill 금지 |
| **9340** | claude-commander (`start.sh`) | — | ✅ 자유롭게 제어 가능 |
| **9341** | 쿠팡 광고센터 Chrome (coupang_ads 프로필) | `chrome-devtools-9341` | ✅ 자유롭게 제어 가능 |
| **9222** | 일반 Chrome (coq3820 프로필) | `chrome-devtools-9222` | ✅ 자유롭게 제어 가능 |

---

## 멀티 프로젝트 포트 충돌 방지

여러 Electron 프로젝트를 동시에 띄울 때 포트가 겹치면 두 번째 앱이 CDP에 접근 불가.

**포트 할당 규칙:**
- `9333` → web-editor 서브 렌더러 — 변경 금지
- `9334` → web-editor (sangpe-editor) 메인 `dev` — 변경 금지
- `9335` → web-editor `dev:step2` / go-finder
- `9336` → web-editor `dev:planning` (goditor admin 모드)
- `9337` → web-editor `dev:ui-polish` (goditor 피그마)
- `9338` → web-editor `dev:design-token`
- `9339` → web-editor `dev:template-tag`
- `9340` → claude-commander (`start.sh` 고정)
- `9341` → 쿠팡 광고센터 Chrome (coupang_ads 프로필) — `/chrome-cdp -coupang-ads`
- `9342+` → 신규 프로젝트 자유 배정

**다른 프로젝트 세션에 전달할 내용:**
> "이 프로젝트 Electron 앱은 `--remote-debugging-port=9341` 이상 포트로 띄워줘. 9333~9340은 기존 앱 전용이야."

**go-finder `package.json` 설정 예시:**
```json
"app:dev": "ELECTRON_DEV=1 electron . --remote-debugging-port=9335"
```

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
