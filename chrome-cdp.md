---
name: chrome-cdp
description: Chrome CDP(Remote Debugging) 포트를 열고 MCP chrome-devtools로 브라우저를 제어하는 스킬이야. 사용자가 "CDP 열어줘", "크롬 디버깅 포트 열어줘", "CDP 연결해줘", "chrome-devtools 써줘" 등의 요청을 할 때 실행해.
---

# Chrome CDP 디버깅 포트 관리

MCP `chrome-devtools` 툴을 사용하기 위해 Chrome을 CDP 모드로 실행하는 스킬.

## 기본 설정

- **포트**: `9222`
- **user-data-dir 기본값**: `/tmp/chrome-debug` (격리된 임시 프로필)

## 실행 순서

### 1단계: CDP 상태 확인

아래 명령어로 포트가 열려 있는지 확인:

```bash
curl -s http://localhost:9222/json/version 2>/dev/null | python3 -c "
import sys, json
try:
    data = json.load(sys.stdin)
    print('READY:', data.get('webSocketDebuggerUrl', ''))
except:
    print('NO_CDP')
"
```

- `READY:` 출력 → 이미 열려 있음, 3단계로 이동
- `NO_CDP` → 2단계 실행

### 2단계: Chrome CDP 모드로 실행

```bash
# a1 유저의 기존 CDP Chrome 종료
pkill -u a1 -f "remote-debugging-port=9222" 2>/dev/null
sleep 1

# CDP 모드로 재시작
"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" \
  --remote-debugging-port=9222 \
  --remote-allow-origins='*' \
  --user-data-dir=/tmp/chrome-debug \
  --no-first-run \
  --no-default-browser-check \
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

## 특정 계정 세션 사용 시

로그인 세션이 필요하면 user-data-dir 변경:

| 계정 | user-data-dir |
|------|---------------|
| 쿠팡 Wing (bnam91) | `/Users/a1/Documents/github_cloud/user_data/coupangWing_bnam91` |
| 임시/격리 (기본값) | `/tmp/chrome-debug` |

user-data-dir만 바꿔서 2단계 재실행.

## 주의사항

- `desk00` 유저의 Chrome은 `pkill -u a1`으로 건드리지 않음
- `/tmp/chrome-debug`는 Google 로그인 없는 클린 상태
- Chrome 재시작 시 기존 탭 세션 초기화됨
