---
name: chrome-cdp
description: Chrome CDP(Remote Debugging) 포트를 열고 MCP chrome-devtools로 브라우저를 제어하는 스킬이야. 사용자가 "CDP 열어줘", "크롬 디버깅 포트 열어줘", "CDP 연결해줘", "chrome-devtools 써줘" 등의 요청을 할 때 실행해.
---

# Chrome CDP 디버깅 포트 관리

MCP `chrome-devtools` 툴을 사용하기 위해 Chrome을 CDP 모드로 실행하는 스킬.

## 프로필 별칭 (인수)

`/chrome-cdp` 뒤에 `-별칭`을 붙이면 해당 계정 세션으로 CDP를 실행한다.

| 포트 | 인수 | 계정 | user-data-dir | 비고 |
|------|------|------|---------------|------|
| 9222 | (없음) | 격리/임시 | `/tmp/chrome-debug` | 로그인 없는 클린 상태 |
| 9222 | `-coq3820` | coq3820@gmail.com (현빈개인) | `/Users/a1/github/user_data/google_coq3820` | 구글 로그인 유지됨 |
| 9222 | `-bnam91` | bnam91@goyamkt.com (고야) | `/Users/a1/github/user_data/naver_bnam91` | — |
| 9290 | _(미설정)_ | — | — | 프로필 슬롯 1 |
| 9291 | _(미설정)_ | — | — | 프로필 슬롯 2 |
| 9292 | _(미설정)_ | — | — | 프로필 슬롯 3 |

- 기본 포트는 `9222`. 여러 프로필을 동시에 열려면 다른 포트 슬롯 사용
- user_data 기본 경로: `/Users/a1/github/user_data/`

> **주의**: Chrome 쿠키는 macOS Keychain 키로 암호화되어 있어 프로필 복사 방식은 Google 세션이 작동하지 않음. 각 프로필 디렉토리에서 직접 로그인한 세션만 사용 가능.

### 프로필 선택 로직

```bash
# 인수 파싱 예시: /chrome-cdp -coq3820
ALIAS="coq3820"  # 인수에서 추출

case "$ALIAS" in
  coq3820)
    USER_DATA_DIR="~/Documents/github_cloud/user_data/google_coq3820"
    ;;
  bnam91)
    USER_DATA_DIR="~/Documents/github_cloud/user_data/coupangWing_bnam91"
    ;;
  *)
    USER_DATA_DIR="/tmp/chrome-debug"
    ;;
esac
```

## 기본 설정

- **포트**: `9222`
- **user-data-dir 기본값**: `/tmp/chrome-debug` (격리된 임시 프로필)

## 실행 순서

### 1단계: CDP 상태 및 프로필 확인

포트가 열려 있어도 **현재 실행 중인 프로필이 요청한 것과 다르면 재시작**해야 한다.

**먼저 non-CDP Chrome 확인** — 같은 프로필로 일반 Chrome이 실행 중이면 프로필 잠금 충돌이 발생한다. kill 전에 반드시 확인하고 사용자에게 알려야 한다.

```bash
TARGET_DIR=$(eval echo "$USER_DATA_DIR")

# 1-A. non-CDP Chrome이 같은 프로필로 실행 중인지 확인
NON_CDP=$(ps aux | grep "Google Chrome" | grep -v grep | grep -v "remote-debugging-port" | grep "$TARGET_DIR" | head -1)
if [ -n "$NON_CDP" ]; then
  echo "⚠️ a1 유저의 Chrome이 non-CDP로 실행 중 ($TARGET_DIR, port 없음)"
  echo "→ kill 후 Preferences 초기화 → CDP 재시작합니다."
  # 사용자에게 알리고 2단계로 진행 (kill 대상: non-CDP Chrome)
  pkill -u a1 -f "Google Chrome" 2>/dev/null
  sleep 1
fi

# 1-B. CDP Chrome 상태 확인
CURRENT_PROFILE=$(ps aux | grep "remote-debugging-port=9222" | grep -v grep | grep -o "\-\-user-data-dir=[^ ]*" | cut -d= -f2)

if curl -s http://localhost:9222/json/version >/dev/null 2>&1; then
  if [ "$CURRENT_PROFILE" = "$TARGET_DIR" ]; then
    echo "✅ 이미 올바른 프로필로 CDP 실행 중 → 3단계로"
    # SKIP_RESTART=true
  else
    echo "⚠️ 다른 프로필로 실행 중 (현재: $CURRENT_PROFILE)"
    echo "→ $TARGET_DIR 로 재시작"
    # SKIP_RESTART=false → 2단계 실행
  fi
else
  echo "CDP 미실행 → 2단계 실행"
fi
```

- non-CDP Chrome 실행 중 → **사용자에게 알리고** kill 후 2단계
- 동일 프로필 CDP 실행 중 → **3단계로 바로 이동**
- 다른 프로필 or 미실행 → **2단계 실행**

### 2단계: Chrome CDP 모드로 실행

> **⚠️ kill은 충돌 시에만** — 1단계에서 이미 non-CDP 충돌이나 다른 프로필인 경우만 여기 도달.
> MCP가 연결 중인 상태에서 kill하면 WebSocket이 끊겨 MCP 서버가 종료됨 → CC 재시작 필요.
> 따라서 **1단계에서 "동일 프로필 CDP 실행 중"이면 절대 kill하지 않고 3단계로 바로 이동.**

```bash
# kill이 필요한 경우만 실행 (non-CDP 충돌 or 다른 프로필)
# SIGTERM 먼저 시도
pkill -u a1 -f "remote-debugging-port=9222" 2>/dev/null
sleep 1
# 포트 여전히 점유 중이면 kill -9 (이 경우 MCP 서버도 죽으므로 CC 재시작 필요)
if lsof -ti :9222 >/dev/null 2>&1; then
  lsof -ti :9222 | xargs kill -9 2>/dev/null
  sleep 1
  echo "⚠️ kill -9 사용됨 → CDP 작업 후 CC 재시작 필요"
fi

# 크래시 복원 팝업 방지: 프로필 종료 상태를 "정상 종료"로 초기화
PREF_FILE="${USER_DATA_DIR:-/tmp/chrome-debug}/Default/Preferences"
if [ -f "$PREF_FILE" ]; then
  python3 -c "
import json
with open('$PREF_FILE', 'r') as f:
    prefs = json.load(f)
prefs.setdefault('profile', {})['exit_type'] = 'Normal'
prefs.setdefault('profile', {})['exited_cleanly'] = True
with open('$PREF_FILE', 'w') as f:
    json.dump(prefs, f)
" 2>/dev/null
fi

# CDP 모드로 재시작 (USER_DATA_DIR 변수 사용)
arch -arm64 "/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" \
  --remote-debugging-port=9222 \
  --remote-allow-origins='*' \
  --user-data-dir="${USER_DATA_DIR:-/tmp/chrome-debug}" \
  --no-first-run \
  --no-default-browser-check \
  --hide-crash-restore-bubble \
  --disable-blink-features=AutomationControlled \
  > /tmp/chrome_cdp.log 2>&1 &

# 포트 열릴 때까지 대기 (최대 10초)
python3 - << 'EOF'
import time, urllib.request, json

for i in range(10):
    try:
        with urllib.request.urlopen('http://localhost:9222/json/version', timeout=2) as r:
            data = json.loads(r.read())
            print(f"✅ CDP 준비 완료: {data.get('webSocketDebuggerUrl')}")
            break
    except:
        print(f"  대기 중... ({i+1}/10)")
        time.sleep(1)
else:
    print("❌ CDP 포트 열기 실패. /tmp/chrome_cdp.log 확인")
EOF
```

### 3단계: MCP chrome-devtools로 제어

포트가 열리면 MCP 툴 사용:

```python
# 열린 탭 목록 확인
mcp__chrome-devtools__list_pages()

# URL 이동
mcp__chrome-devtools__navigate_page(type="url", url="https://www.naver.com")

# 스크린샷
mcp__chrome-devtools__take_screenshot()

# JS 실행
mcp__chrome-devtools__evaluate_script(script="document.title")
```

## 주의사항 (프로필 관리)

- 각 프로필은 해당 디렉토리에서 직접 로그인한 세션을 그대로 사용
- 세션 만료 시 CDP Chrome 창에서 직접 재로그인 필요
- 단, Google 계정은 CDP 모드에서 신규 OAuth 로그인 차단됨 → 기존 세션 유지 중에만 사용 가능

## MCP vs WebSocket

MCP `chrome-devtools`는 기본적으로 자체 Chrome을 띄움. 포트 9222(로그인된 프로필)에 연결하려면:
- `~/.claude.json`의 chrome-devtools 설정에 `--browserUrl http://127.0.0.1:9222` 인수 필요
- 설정 변경 후 **Claude Code 재시작** 필요

재시작 없이 즉시 사용하려면 Python WebSocket으로 직접 제어:
```python
import websocket, json, urllib.request

with urllib.request.urlopen('http://localhost:9222/json/version') as r:
    ws_url = json.loads(r.read())['webSocketDebuggerUrl']

ws = websocket.create_connection(ws_url)
# Target.createTarget으로 새 탭 열기
ws.send(json.dumps({"id":1,"method":"Target.createTarget","params":{"url":"https://example.com"}}))
```

## 캡챠 처리

페이지 이동 후 캡챠가 감지되면 **사용자에게 묻지 말고 직접 해결 시도**:

1. [*NAVER*로그인](https://nid.naver.com/nidlogin.login?mode=form&url=https://www.naver.com/)`mcp__chrome-devtools__take_screenshot()` 으로 캡챠 화면 캡처
2. 이미지에서 질문 텍스트 읽기 (예: "가게 전화번호의 뒤에서 1번째 숫자는?")
3. 이미지 내 정답 확인 후 `mcp__chrome-devtools__fill()` 또는 `mcp__chrome-devtools__click()` 으로 입력
4. 확인 버튼 클릭 후 `mcp__chrome-devtools__take_screenshot()` 으로 결과 확인
5. 캡챠 통과 실패 시에만 사용자에게 안내

## 주의사항

- `desk00` 유저의 Chrome은 `pkill -u a1`으로 건드리지 않음
- `/tmp/chrome-debug`는 Google 로그인 없는 클린 상태
- Chrome 재시작 시 기존 탭 세션 초기화됨

## ⚠️ 일렉트론 앱 포트 절대 금지

**포트 9333, 9334는 일렉트론 앱(web-editor) 전용이다. 절대 건드리지 않는다.**

- `pkill` 등 프로세스 종료 시 `remote-debugging-port=9222` 대상만 종료
- `remote-debugging-port=9333` 또는 `9334` 프로세스는 어떤 경우에도 kill 금지
- chrome-devtools MCP가 9333/9334 탭을 반환해도 일렉트론 앱 탭이므로 navigate/click 등 조작 금지
- 일반 Chrome 브라우저는 반드시 **포트 9222** 만 사용
