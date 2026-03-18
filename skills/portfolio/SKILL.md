---
name: portfolio
description: |
  이력서를 기반으로 Gamma API를 활용해 포트폴리오 웹페이지를 자동 생성합니다.
  취업 준비생이 이력서를 붙여넣거나 파일로 제공하면, 전문적인 포트폴리오 웹페이지/프레젠테이션/문서를 만들어줍니다.
  사용법: /portfolio 입력 후 안내에 따라 이력서를 제공하세요.
allowed-tools:
  - Bash
  - Read
  - Write
  - AskUserQuestion
---

# Portfolio Generator (Gamma API)

이력서를 기반으로 Gamma API를 활용해 전문적인 포트폴리오를 자동 생성하는 스킬입니다.

## Step 1: API 키 확인 및 설정

```bash
echo "${GAMMA_API_KEY:+set}"
```

**API 키가 설정되어 있지 않은 경우**, 사용자에게 안내합니다:

> Gamma API 키가 필요합니다.
>
> 1. https://gamma.app > 프로필 > **Account Settings** > **API Keys** > **Create API Key**
> 2. **필요 플랜:** Pro, Ultra, Teams, 또는 Business
>
> 설정 방법: `export GAMMA_API_KEY="발급받은_키"`

AskUserQuestion으로 API 키 입력을 받습니다. Other 란에 직접 입력한 경우 환경변수로 설정합니다.
**중요: API 키를 절대 화면에 출력하거나 로그에 남기지 마세요.**

## Step 2: 이력서 수집

AskUserQuestion으로 이력서 입력 방식을 물어봅니다:

1. **텍스트 붙여넣기** — 다음 메시지에 이력서 텍스트를 직접 붙여넣기
2. **파일 경로 제공** — 이력서 파일(.txt, .md, .pdf) 경로를 입력

이력서 내용이 100자 미만이면 다시 요청합니다.

## Step 3: 포트폴리오 옵션 선택

AskUserQuestion으로 2개의 질문을 동시에 물어봅니다:

**질문 1 — 출력 형식:**
- 웹페이지 (Recommended)
- 프레젠테이션
- 문서

**질문 2 — 스타일:**
- 전문적 (Recommended)
- 캐주얼
- 창의적

## Step 4: Gamma API 호출

### 파라미터 매핑

**format 매핑:**

| 선택 | format |
|------|--------|
| 웹페이지 | `webpage` |
| 프레젠테이션 | `presentation` |
| 문서 | `document` |

**스타일 → tone + themeId 매핑:**

| 선택 | tone | themeId |
|------|------|---------|
| 전문적 | `professional, confident` | `commons` |
| 캐주얼 | `casual, friendly` | `gamma` |
| 창의적 | `creative, bold` | `electric` |

### API 호출

python3을 사용하여 JSON을 안전하게 구성하고 호출합니다:

```bash
python3 -c "
import json, subprocess, sys

input_text = '''다음 이력서를 기반으로 포트폴리오를 만들어주세요.

섹션 구성:

히어로: 이름, 직함, 한줄 소개, 연락처 링크
---
About Me: 핵심 역량 3가지
---
경력사항: 타임라인 형태
---
주요 프로젝트: 카드 형태, 성과 중심
---
Expertise: 핵심 키워드 나열
---
교육 및 자격
---
연락처

이력서:
---
RESUME_CONTENT_HERE'''

additional = '''텍스트는 간결하게 작성해주세요. em dash, 가운뎃점, 이모지는 사용하지 마세요. 텍스트와 시각 요소의 균형을 맞춰주세요.'''

payload = {
    'inputText': input_text,
    'additionalInstructions': additional,
    'textMode': 'generate',
    'format': 'FORMAT_HERE',
    'themeId': 'THEME_HERE',
    'cardSplit': 'inputTextBreaks',
    'textOptions': {
        'tone': 'TONE_HERE',
        'audience': 'hiring managers and recruiters',
        'amount': 'medium',
        'language': 'ko'
    },
    'imageOptions': {
        'source': 'aiGenerated',
        'style': 'minimal abstract, clean'
    },
    'exportAs': 'pdf'
}

result = subprocess.run(
    ['curl', '-s', '-w', '\\n%{http_code}', '-X', 'POST',
     'https://public-api.gamma.app/v1.0/generations',
     '-H', 'X-API-KEY: ' + sys.argv[1],
     '-H', 'Content-Type: application/json',
     '-d', json.dumps(payload)],
    capture_output=True, text=True
)
print(result.stdout)
" "\$GAMMA_API_KEY"
```

**RESUME_CONTENT_HERE**, **FORMAT_HERE**, **THEME_HERE**, **TONE_HERE** 를 Step 3에서 선택된 값으로 교체합니다.

### 응답 처리

응답에서 `generationId`를 추출합니다. 에러 시:
- 401: "API 키가 유효하지 않습니다."
- 402: "크레딧이 부족합니다."
- 429: "요청이 너무 많습니다. 잠시 후 다시 시도해주세요."

## Step 5: 폴링 및 결과 확인

"포트폴리오를 생성하고 있습니다... (보통 30초~2분 소요)" 메시지 표시 후, 5초 간격으로 폴링합니다.

```bash
for i in $(seq 1 60); do
  sleep 5
  STATUS_RESPONSE=$(curl -s -X GET "https://public-api.gamma.app/v1.0/generations/${GENERATION_ID}" \
    -H "X-API-KEY: ${GAMMA_API_KEY}")
  STATUS=$(echo "$STATUS_RESPONSE" | python3 -c "import sys,json; print(json.load(sys.stdin).get('status','unknown'))")

  if [ "$STATUS" = "completed" ]; then
    GAMMA_URL=$(echo "$STATUS_RESPONSE" | python3 -c "import sys,json; print(json.load(sys.stdin).get('gammaUrl',''))")
    EXPORT_URL=$(echo "$STATUS_RESPONSE" | python3 -c "import sys,json; print(json.load(sys.stdin).get('exportUrl',''))")
    echo "COMPLETED"
    echo "GAMMA_URL=$GAMMA_URL"
    echo "EXPORT_URL=$EXPORT_URL"
    break
  elif [ "$STATUS" = "failed" ]; then
    echo "FAILED"
    echo "$STATUS_RESPONSE"
    break
  fi
done
```

## Step 6: 결과 안내

### 성공 시

```
포트폴리오가 성공적으로 생성되었습니다!

포트폴리오 보기: {gammaUrl}
PDF 다운로드: {exportUrl}

Gamma 에디터에서 텍스트, 이미지, 레이아웃을 자유롭게 수정할 수 있습니다.
```

### 실패 시

에러 원인을 분석하여 안내하고 재시도 여부를 물어봅니다.

### 타임아웃 시 (5분 초과)

```
Generation ID: {generationId}
나중에 https://gamma.app 내 문서에서 확인할 수 있습니다.
```

## 에러 처리 요약

| 상황 | 대응 |
|------|------|
| API 키 미설정 | 설정 가이드 안내 |
| 401 | 키 재확인 안내 |
| 402 | Gamma 대시보드에서 크레딧 충전 안내 |
| 429 | 잠시 후 재시도 안내 |
| 이력서 부족 | 이력서 보강 안내 |
| 생성 실패 | 에러 분석 + 재시도 제안 |

## 참고사항

- Gamma Pro, Ultra, Teams, 또는 Business 플랜 필요
- 생성 1회당 Gamma 크레딧 소모
- API 키: https://gamma.app > Account Settings > API Keys
- 생성된 포트폴리오는 Gamma 에디터에서 편집 가능
- export URL은 약 7일 후 만료
