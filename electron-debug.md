---
name: electron-debug
description: 상페마법사 웹에디터 Electron 앱 디버깅 환경을 확인하고 DevTools 접속 정보를 제공하는 스킬. 사용자가 "일렉트론 디버깅", "앱 디버깅 연결", "DevTools 열어줘", "electron devtools" 등을 말할 때 실행해.
---

# 상페마법사 웹에디터 Electron 디버깅

## 앱 정보

- **프로젝트 경로**: `/Users/a1/web-editor`
- **실행 명령**: `cd /Users/a1/web-editor && npm run dev`
- **디버깅 포트**: `9334` (렌더러), `9333` (서브 렌더러)
- **dev 스크립트**: `electron . --enable-logging --remote-debugging-port=9334 --remote-allow-origins=* admin`

## ⚠️ 포트 보호 규칙

**포트 9333, 9334는 Electron 앱 전용. 절대 kill/종료 금지.**
일반 Chrome CDP는 반드시 **포트 9222** 사용.

## 실행 순서

### 1단계: 앱 실행 상태 확인

```bash
# Electron 앱 실행 중인지 확인
ps aux | grep "remote-debugging-port=9334" | grep -v grep

# 포트 응답 확인
curl -s http://localhost:9334/json/version
```

- 실행 중 → 2단계로
- 미실행 → 앱 먼저 시작: `cd /Users/a1/web-editor && npm run dev`

### 2단계: 렌더러 탭 목록 조회

```bash
curl -s http://localhost:9334/json | python3 -c "
import json, sys
tabs = json.load(sys.stdin)
for t in tabs:
    print(f\"[{t.get('type')}] {t.get('title')} — {t.get('url')}\")
    dfu = t.get('devtoolsFrontendUrl')
    if dfu:
        print(f\"  DevTools: http://localhost:9334{dfu}\")
"
```

### 3단계: DevTools 접속

`devtoolsFrontendUrl`을 Chrome에서 열면 풀 DevTools 사용 가능:
- 브레이크포인트 / 스텝 디버깅
- 콜스택 확인
- 소스 패널, 네트워크 패널

Chrome에서 열기:
```bash
open "http://localhost:9334/json"
```

또는 chrome-devtools MCP로 자동 열기:
```python
# MCP chrome-devtools (9334에 연결된 상태)
mcp__chrome-devtools__list_pages()          # 탭 목록
mcp__chrome-devtools__take_screenshot()     # 현재 화면
mcp__chrome-devtools__evaluate_script(script="document.title")  # JS 실행
```

## MCP 연결 구조

| MCP 이름 | 포트 | 대상 |
|----------|------|------|
| `chrome-devtools` | 9334 | Electron 앱 렌더러 |
| `chrome-devtools-9222` | 9222 | 일반 Chrome (coq3820 프로필) |

## 자주 쓰는 디버깅 명령

```bash
# 렌더러 콘솔 로그 확인 (enable-logging 덕분에 터미널에도 출력됨)
tail -f /tmp/electron_debug.log 2>/dev/null

# 앱 재시작 (포트 유지)
pkill -f "remote-debugging-port=9334" && cd /Users/a1/web-editor && npm run dev
```
