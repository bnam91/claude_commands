ImgBB에 이미지를 업로드하는 스킬이야. 로컬에서 사용하는 방법이야.
스킬 경로: ~/Documents/github_cloud/utils_mac/utils_imgbb

## 기능 요약

- **단일 업로드** - 파일 선택 다이얼로그 또는 파일 경로 직접 지정
- **복수 업로드** - 폴더 경로 지정 또는 JSON 설정
- 업로드 완료 시 URL을 **클립보드에 자동 복사**
- 업로드된 URL을 **Google Sheets에 자동 저장**

---

## 단일 업로드

### GUI 다이얼로그로 파일 선택
```bash
cd ~/Documents/github_cloud/utils_mac/utils_imgbb && node scripts/imgbb-upload.js
```
- 파일 선택 창이 열림 (복수 선택 가능)

### 파일 경로 직접 지정
```bash
cd ~/Documents/github_cloud/utils_mac/utils_imgbb && node scripts/imgbb-upload.js /경로/이미지.png
```

---

## 복수 업로드

### 폴더 경로 입력 방식 (터미널에서 입력)
```bash
cd ~/Documents/github_cloud/utils_mac/utils_imgbb && npm run many
```
또는
```bash
cd ~/Documents/github_cloud/utils_mac/utils_imgbb && node scripts/imgbb-upload_many.js
```
실행 후 폴더 경로를 입력하면 해당 폴더의 이미지 전체 업로드

### 폴더 경로 JSON에 미리 지정
`scripts/imgbb-upload_many.json` 의 `folder` 항목에 경로 입력 후 실행:
```json
{
  "folder": "/Users/a1/Desktop/업로드할폴더",
  "spreadsheetId": "1kmOA_YdI-GU0g01ZuCLTg4ae8bbvhnd7pvjAwawHMO4",
  "sheetName": "시트1",
  "filenameWithoutExt": true
}
```

### 파일 경로 직접 나열
```bash
cd ~/Documents/github_cloud/utils_mac/utils_imgbb && node scripts/imgbb-upload_many.js /경로/1.png /경로/2.png /경로/3.png
```

---

## 업로드 결과

- 각 이미지 업로드 완료 시 URL이 **클립보드에 복사**됨 (마지막 URL이 최종 저장)
- Google Sheets에 파일명 + URL 목록 자동 저장
  - 저장 시트: https://docs.google.com/spreadsheets/d/1kmOA_YdI-GU0g01ZuCLTg4ae8bbvhnd7pvjAwawHMO4

---

## 설정

- **API 키**: `~/Documents/github_cloud/module_api_key/.env` 의 `IMGBB_API_KEY` 자동 로드
- **지원 확장자**: png, jpg, jpeg, gif, webp, bmp
- **파일 정렬**: 자연 정렬 (101, 102, 200, 301 ... 순)
- **filenameWithoutExt**: true 시 확장자 제외한 파일명으로 시트 저장

---

---

## Google Drive 폴더 → ImgBB (Drive 링크 전달 시)

Drive 폴더 링크를 받으면 아래 흐름으로 처리해.

**고정 스프레드시트**: `1kmOA_YdI-GU0g01ZuCLTg4ae8bbvhnd7pvjAwawHMO4`

### 처리 흐름

**Step 1. 폴더명 조회 + 이미지 다운로드 (Python)**
```python
import sys, io, os
sys.path.insert(0, str(__import__('pathlib').Path.home() / 'Documents/github_cloud/module_auth'))
from auth import get_credentials
from googleapiclient.discovery import build
from googleapiclient.http import MediaIoBaseDownload

creds = get_credentials()
drive = build('drive', 'v3', credentials=creds)

folder_id = '추출한_폴더_ID'
folder_name = drive.files().get(fileId=folder_id, fields='name').execute()['name']

IMAGE_MIMES = ['image/png','image/jpeg','image/jpg','image/gif','image/webp','image/bmp']
files = drive.files().list(
    q=f"'{folder_id}' in parents and trashed=false",
    fields='files(id, name, mimeType)', pageSize=100
).execute().get('files', [])
images = [f for f in files if f.get('mimeType','') in IMAGE_MIMES]

MIME_EXT = {'image/png': '.png', 'image/jpeg': '.jpg', 'image/gif': '.gif', 'image/webp': '.webp', 'image/bmp': '.bmp'}
tmp_dir = f'/tmp/{folder_name}'
os.makedirs(tmp_dir, exist_ok=True)
for img in images:
    name = img['name']
    ext = MIME_EXT.get(img['mimeType'], '.png')
    # 확장자 없거나 잘못된 경우 강제 부여
    if not any(name.lower().endswith(e) for e in MIME_EXT.values()):
        name = name + ext
    save_path = f"{tmp_dir}/{name}"
    request = drive.files().get_media(fileId=img['id'])
    fh = io.FileIO(save_path, 'wb')
    MediaIoBaseDownload(fh, request).next_chunk()
    fh.close()
print(f"다운로드 완료: {tmp_dir} ({len(images)}개)")
```

**Step 2. 시트 탭 생성 (없으면 추가)**
```python
from googleapiclient.discovery import build
sheets = build('sheets', 'v4', credentials=creds)
spreadsheet_id = '1kmOA_YdI-GU0g01ZuCLTg4ae8bbvhnd7pvjAwawHMO4'

existing = sheets.spreadsheets().get(spreadsheetId=spreadsheet_id).execute()
tab_names = [s['properties']['title'] for s in existing['sheets']]
if folder_name not in tab_names:
    sheets.spreadsheets().batchUpdate(
        spreadsheetId=spreadsheet_id,
        body={'requests': [{'addSheet': {'properties': {'title': folder_name}}}]}
    ).execute()
    print(f"시트 탭 생성: {folder_name}")
```

**Step 3. imgbb-upload_many.json 업데이트 후 업로드**
```python
import json
json_path = os.path.expanduser('~/Documents/github_cloud/utils_mac/utils_imgbb/scripts/imgbb-upload_many.json')
with open(json_path) as f:
    old_config = json.load(f)

new_config = {**old_config, 'folder': tmp_dir, 'sheetName': folder_name}
with open(json_path, 'w') as f:
    json.dump(new_config, f, indent=2, ensure_ascii=False)
```
```bash
cd ~/Documents/github_cloud/utils_mac/utils_imgbb && node scripts/imgbb-upload_many.js
```

**Step 4. JSON 복원 + 임시 파일 삭제**
```python
with open(json_path, 'w') as f:
    json.dump(old_config, f, indent=2, ensure_ascii=False)
import shutil
shutil.rmtree(tmp_dir)
print("임시 파일 삭제 완료")
```

### 이미지 1개인 경우 (MIN_FILES=2 제한 우회)
`imgbb-upload_many.js`는 최소 2개 필요. 1개면 단일 업로드 후 시트에 직접 저장:
```bash
cd ~/Documents/github_cloud/utils_mac/utils_imgbb && node scripts/imgbb-upload.js "/tmp/파일명"
```
이후 Python으로 시트에 직접 `values.update` 호출.

---

## 사용자 요청 처리

- "이미지 업로드해줘" → 단일/복수 중 확인 후 명령어 안내
- "폴더 업로드해줘" → npm run many 실행 안내 또는 직접 실행
- "URL 어디 저장돼?" → Google Sheets URL 안내
- "설정 바꾸고 싶어" → imgbb-upload_many.json 수정 안내
- Drive 폴더 링크 전달 시 → Drive → ImgBB 변환 흐름 실행 (폴더명으로 시트 탭 생성, 업로드 후 /tmp 삭제)
