구글 시트의 텍스트 셀을 HTML로 변환해서 출력 셀에 저장해줘.

## Step 1: 기본 HTML 변환 (Python)

아래 명령어를 실행하고 결과를 그대로 출력해줘:

```bash
python3 ~/Documents/claude_skills/html_converter/html_converter.py
```

## Step 2: 포인트 색상 적용 (Claude 직접)

Step 1 완료 후, 아래 명령어로 C3 / E3 / G3 셀을 읽어줘:

```bash
python3 ~/Documents/claude_skills/html_converter/cell_read.py C3
python3 ~/Documents/claude_skills/html_converter/cell_read.py E3
python3 ~/Documents/claude_skills/html_converter/cell_read.py G3
```

읽은 HTML을 분석해서 **제품명**과 **제품의 핵심 특징이 드러나는 문구**를 찾아 색상을 입혀줘.

**색상 적용 지침:**
- C3 / E3 / G3 각각 **서로 다른 색상 조합** 사용 (획일적 금지)
- 허용 스타일:
  - `background-color: rgb(161, 208, 255)` (연파랑 배경)
  - `color: rgb(0, 123, 217)` (파랑 글자)
  - `font-weight: bold`
- 적용 위치: 제품명 / 재료·식감·맛 등 핵심 특징 문구 / 강조할 키워드
- ■ 섹션 헤더나 일정 정보에는 색상 넣지 않음

수정한 HTML은 아래 방식으로 저장해줘:

```bash
python3 /tmp/apply_colors.py
```

단, apply_colors.py는 매번 새로 작성해서 실행할 것 (셀 내용이 바뀌기 때문).

두 단계 완료 후, 각 셀에 어떤 문구에 색상을 입혔는지 간략히 요약해줘.
