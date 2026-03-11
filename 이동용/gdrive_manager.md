Google Drive를 인덱스 아키텍처 규칙에 맞춰 관리하는 스킬이야. 폴더/파일 조회 및 검색을 할 수 있어.
스킬 경로: ~/Documents/claude_skills/gdrive_manager

## 인덱스 아키텍처 v1.2

| 대역 | 역할 |
|------|------|
| 000xx | 공지 / 로그 / 릴리즈 |
| 100xx | 컨테이너 (개별박스, 히스토리, expired) |
| 200xx | 모듈 & 유틸 & 에셋 |
| 300xx | 프로젝트 메인 |
| 500xx | dev |
| 800xx | 외부 공유 |
| 900xx | 긴급요청 |

---

## 등록된 드라이브

| 별칭 | 폴더 ID |
|------|---------|
| 꽃킬커머스 | 1PwF-ko5-qWWXIsBePBq3EYEV5vxfBbZd |

---

## 명령어

### 드라이브 목록
```bash
cd ~/Documents/claude_skills/gdrive_manager && python3 gdrive_manager.py --list-drives
```

### 드라이브 추가
```bash
cd ~/Documents/claude_skills/gdrive_manager && python3 gdrive_manager.py --add-drive --alias 별칭 --folder-id 폴더ID
```

### 폴더 내용 조회
```bash
# 루트 조회
cd ~/Documents/claude_skills/gdrive_manager && python3 gdrive_manager.py --list --drive 꽃킬커머스

# 하위 폴더 조회
cd ~/Documents/claude_skills/gdrive_manager && python3 gdrive_manager.py --list --drive 꽃킬커머스 --folder 300
```

### 파일 검색
```bash
cd ~/Documents/claude_skills/gdrive_manager && python3 gdrive_manager.py --search --drive 꽃킬커머스 --query 파일명
```

### 인덱스 아키텍처 보기
```bash
cd ~/Documents/claude_skills/gdrive_manager && python3 gdrive_manager.py --index
```

---

## Drive → ImgBB URL 변환

Drive 폴더 URL을 받으면 이미지 파일을 ImgBB에 업로드해서 URL을 반환할 수 있어.

### 처리 흐름
1. URL에서 폴더 ID 추출
2. Drive API로 이미지 파일 목록 조회 (png/jpg/jpeg/gif/webp/bmp)
3. 이미지를 `/tmp/`에 임시 다운로드
4. ImgBB 스크립트로 업로드 → URL 획득
5. `/tmp/` 임시 파일 즉시 삭제

### ImgBB 업로드 명령어
```bash
cd ~/Documents/github_cloud/utils_mac/utils_imgbb && node scripts/imgbb-upload.js "/tmp/파일명"
```

업로드 후 반드시 임시 파일 삭제:
```bash
rm "/tmp/파일명"
```

---

## 사용자 요청 처리

- "꽃킬커머스 드라이브 보여줘" → --list --drive 꽃킬커머스 실행
- "300 폴더 보여줘" → --list --drive 꽃킬커머스 --folder 300 실행
- "파일 찾아줘" → --search --drive 드라이브명 --query 검색어 실행
- "드라이브 추가해줘" → 링크에서 폴더 ID 추출 후 --add-drive 실행
- "인덱스 규칙 알려줘" → --index 실행 또는 위 표 보여주기
- "이 드라이브 폴더 imgbb url 만들어줘" → Drive → ImgBB 변환 흐름 실행 (업로드 후 /tmp 파일 삭제)

결과는 한국어로 정리해서 보여줘.
