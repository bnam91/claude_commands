구글 시트 '고야내부용'에 입금요청을 등록하는 스킬이야.
스크립트 경로: ~/Documents/claude_skills/payment_request/payment_request.py

## 시트 정보
- 스프레드시트 ID: 1CK2UXTy7HKjBe2T0ovm5hfzAAKZxZAR_ev3cbTPOMPs
- 시트명: 고야내부용
- 열 구성: E=항목, F=받는사람, I=계좌번호, J=주민/사업자번호, K=금액, P=상태(입금요청)

## 자주쓰는 곳 (favorites.json)
등록된 별칭: 아이플 → 항목=아이플726호, 받는사람=김성모, 계좌=농협 301-0192-8356-11 김성모
favorites.json 경로: ~/Documents/claude_skills/payment_request/favorites.json

## 입력 파싱 규칙

사용자 입력 예시:
- "아이플 관리비 381,430 입금 등록해줘"
- "김철수에게 서비스비 500,000 입금 요청 등록"
- "고야앤드미디어 광고비 1,200,000 등록해줘"

파싱:
- 받는사람/별칭(recipient): 금액 앞에 오는 사람/회사/별칭
- 항목(item): 받는사람 다음에 오는 항목명 (관리비, 서비스비, 급여, 광고비 등)
- 금액(amount): 숫자 + 콤마 조합. 그대로 전달.

**파싱이 불명확한 경우**: 사용자에게 확인 후 진행.

## Step 1: 기존 계좌정보 조회

```bash
python3 ~/Documents/claude_skills/payment_request/payment_request.py \
  --lookup --recipient [받는사람/별칭]
```

결과 형식:
- `FOUND|계좌번호|주민번호|항목템플릿|실제수취인|출처` → Step 2로 바로 진행
- `NOT_FOUND` → Step 1-1로 이동

FOUND일 때: 파이프(|)로 split해서 각 필드 추출.

### Step 1-1: 계좌정보 없을 때

사용자에게 요청:
> "**[받는사람]**의 등록된 계좌 정보가 없습니다. 계좌번호를 알려주시면 함께 등록할게요. (모르면 '없음'이라고 해주세요)"

## Step 2: 입금요청 등록

`--recipient`에는 항상 원래 사용자 입력 별칭을 전달 (스크립트가 favorites/시트에서 자동 매핑):

```bash
python3 ~/Documents/claude_skills/payment_request/payment_request.py \
  --recipient "[별칭 또는 이름]" \
  --item "[항목]" \
  --amount "[금액]"
```

계좌를 사용자가 직접 제공한 경우에만 --account, --business-id 추가.

## Step 3: 결과 보고

```
✅ 입금요청 등록 완료
  항목     : [항목]
  받는사람 : [이름]
  금액     : [금액]
  계좌번호 : [계좌번호]
  상태     : 입금요청
```

## 추가 기능

**입금요청 목록 조회** ("입금요청 목록", "입금 대기 목록" 요청 시):
```bash
python3 ~/Documents/claude_skills/payment_request/payment_request.py --list
```

**자주쓰는 곳 목록** ("자주쓰는 곳 보여줘"):
```bash
python3 ~/Documents/claude_skills/payment_request/payment_request.py --show-favorites
```

**자주쓰는 곳 새로 등록** ("XXX를 자주쓰는 곳에 등록해줘"):
```bash
python3 ~/Documents/claude_skills/payment_request/payment_request.py \
  --add-favorite \
  --alias "[별칭]" \
  --item "[항목템플릿]" \
  --recipient "[받는사람]" \
  --account "[계좌번호]" \
  --business-id "[주민/사업자번호]"  # 선택
```

## 주의사항
- 금액은 입력 그대로 전달 ("381,430" → "381,430")
- 별칭 조회는 favorites → 시트 E열 부분일치 → 시트 F열 정확일치 순
- 모든 결과는 한국어로 정리해서 보여줘
