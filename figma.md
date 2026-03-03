피그마(Figma)를 WebSocket으로 직접 제어하는 스킬이야.

## 실행 환경
- CLI 툴: `node ~/github/util_figma_mcp_bot/figma_cmd.mjs`
- WebSocket 서버: `ws://localhost:3055` (bun으로 실행 중)
- 채널: **스킬 실행 시마다 사용자에게 현재 채널 ID를 물어봐야 함** (매번 바뀜)

## 기본 사용법
```bash
node ~/github/util_figma_mcp_bot/figma_cmd.mjs --command <커맨드> --params '<JSON>'
```

## 주요 커맨드 목록

### 정보 조회
```bash
# 현재 페이지 전체 정보 (레이어 목록 포함)
node ... --command get_document_info --params '{}'

# 특정 노드 상세 정보
node ... --command get_node_info --params '{"nodeId":"1:2"}'

# 현재 선택된 노드
node ... --command get_selection --params '{}'

# 페이지 목록
node ... --command get_pages --params '{}'
```

### 도형 생성
```bash
# 사각형
node ... --command create_rectangle --params '{"x":100,"y":100,"width":200,"height":200,"name":"Box"}'

# 프레임
node ... --command create_frame --params '{"x":0,"y":0,"width":375,"height":812,"name":"Mobile"}'

# 텍스트
node ... --command create_text --params '{"x":100,"y":100,"text":"Hello","name":"Label"}'

# 원형
node ... --command create_ellipse --params '{"x":100,"y":100,"width":100,"height":100}'
```

### 색상/스타일
```bash
# 채우기 색상 (r,g,b,a 모두 0~1 범위) - color 객체로 감싸야 함
node ... --command set_fill_color --params '{"nodeId":"1:2","color":{"r":1,"g":1,"b":1,"a":1}}'

# 테두리 색상 - color 객체로 감싸야 함
node ... --command set_stroke_color --params '{"nodeId":"1:2","color":{"r":0,"g":0,"b":0,"a":1},"strokeWeight":2}'

# 노드 속성 일괄 설정 (opacity, visible, locked 등)
node ... --command set_node_properties --params '{"nodeId":"1:2","opacity":0.5}'
```

### 텍스트
```bash
# 텍스트 내용 변경
node ... --command set_text_content --params '{"nodeId":"1:3","text":"새 텍스트"}'

# 폰트 크기
node ... --command set_font_size --params '{"nodeId":"1:3","fontSize":24}'

# 폰트 굵기
node ... --command set_font_weight --params '{"nodeId":"1:3","fontWeight":700}'
```

### 레이아웃
```bash
# 이동
node ... --command move_node --params '{"nodeId":"1:2","x":200,"y":300}'

# 크기 조정
node ... --command resize_node --params '{"nodeId":"1:2","width":400,"height":300}'

# 이름 변경
node ... --command rename_node --params '{"nodeId":"1:2","name":"새이름"}'

# 삭제
node ... --command delete_node --params '{"nodeId":"1:2"}'

# 자식으로 삽입
node ... --command insert_child --params '{"parentId":"1:2","childId":"1:3"}'
```

### 오토레이아웃
```bash
node ... --command set_auto_layout --params '{"nodeId":"1:2","mode":"VERTICAL","padding":16,"gap":8}'
```

### 모서리 반경
```bash
node ... --command set_corner_radius --params '{"nodeId":"1:2","cornerRadius":8}'
```

## 실행 방법

**스킬 시작 시 반드시 먼저 물어봐야 할 것:**
> "Figma 플러그인에 표시된 채널 ID가 뭐예요? (예: zthk2q2h)"

채널 ID를 받은 뒤 모든 커맨드에 `--channel <채널ID>` 옵션을 붙여서 실행.

사용자 요청을 분석해서 적절한 커맨드를 Bash로 실행해줘.

**노드 ID 파악 방법**: 모르면 먼저 `get_document_info`로 레이어 목록 확인 후 이름으로 매칭.

**여러 작업 순서**: create → set_fill_color → move_node → resize_node 순으로 실행.

**색상 변환**: HEX(#RRGGBB) → r/g/b (각 채널 값 ÷ 255). 예: #FF0000 → r:1, g:0, b:0

**에러 시**: "Timeout: 응답이 없어요" → Figma 플러그인 연결 상태 확인 요청.

결과는 한국어로 정리해서 보여줘.
