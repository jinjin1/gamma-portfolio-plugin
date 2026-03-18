---
name: portfolio
description: |
  Generate a professional portfolio webpage from a resume using the Gamma API.
  Paste or provide a resume file, choose a format and style, and get a polished portfolio in seconds.
  Usage: type /portfolio and follow the prompts.
allowed-tools:
  - Bash
  - Read
  - Write
  - AskUserQuestion
---

# Portfolio Generator (Gamma API)

Generate a professional portfolio from a resume using the Gamma API.

## Step 1: Check API Key

```bash
echo "${GAMMA_API_KEY:+set}"
```

If not set, guide the user:

> You need a Gamma API key.
>
> 1. Go to https://gamma.app > Profile > **Account Settings** > **API Keys** > **Create API Key**
> 2. **Required plan:** Pro, Ultra, Teams, or Business
>
> Set it: `export GAMMA_API_KEY="your_key"`

Use AskUserQuestion to collect the key. If entered in the Other field, set it as an environment variable.
**IMPORTANT: Never print or log the API key.**

## Step 2: Collect Resume

Use AskUserQuestion:

1. **Paste text** — paste resume text in the next message
2. **Provide file path** — path to a resume file (.txt, .md, .pdf)

If the resume is under 100 characters, ask again.

**Language detection:** After receiving the resume, detect its primary language (e.g. Korean, English, Japanese, Chinese). This determines the `textOptions.language` value and the language of the inputText prompt in Step 4.

## Step 3: Choose Options

Use AskUserQuestion with 2 questions:

**Question 1 — Format:**
- Webpage (Recommended)
- Presentation
- Document

**Question 2 — Style:**
- Professional (Recommended)
- Casual
- Creative

## Step 4: Call Gamma API

### Parameter Mapping

**Format:**

| Selection | format |
|-----------|--------|
| Webpage | `webpage` |
| Presentation | `presentation` |
| Document | `document` |

**Style → tone + themeId:**

| Selection | tone | themeId |
|-----------|------|---------|
| Professional | `professional, confident` | `commons` |
| Casual | `casual, friendly` | `gamma` |
| Creative | `creative, bold` | `electric` |

**Language → textOptions.language + inputText prompt:**

| Detected Language | language | inputText language |
|-------------------|----------|--------------------|
| Korean | `ko` | Korean |
| English | `en` | English |
| Japanese | `ja` | Japanese |
| Chinese | `zh` | Chinese |
| Other | ISO 639-1 code | Match resume language |

### API Call

Build the JSON payload and call the API using python3.

The `inputText` prompt must be written in the **detected resume language**. Below are templates for English and Korean. For other languages, translate the English template accordingly.

**English inputText (when resume is in English):**

```
Create a portfolio based on the following resume.

Sections:

Hero: Name, title, one-line summary, contact links
---
About Me: 3 key strengths
---
Experience: Timeline format
---
Key Projects: Card format, results-focused
---
Expertise: List of key skills
---
Education & Certifications
---
Contact

Resume:
---
{resume_content}
```

**Korean inputText (when resume is in Korean):**

```
다음 이력서를 기반으로 포트폴리오를 만들어주세요.

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
{resume_content}
```

**additionalInstructions (same for all languages):**

```
Keep text concise. Do not use em dashes, middle dots, or emojis. Balance text and visual elements.
```

**Full API call:**

```bash
python3 -c "
import json, subprocess, sys

input_text = '''INPUT_TEXT_HERE'''

additional = '''Keep text concise. Do not use em dashes, middle dots, or emojis. Balance text and visual elements.'''

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
        'language': 'LANG_HERE'
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

Replace **INPUT_TEXT_HERE** with the language-appropriate inputText template (with resume inserted), and **FORMAT_HERE**, **THEME_HERE**, **TONE_HERE**, **LANG_HERE** with values from the mapping tables above.

### Response Handling

Extract `generationId` from the response. On error:
- 401: "Invalid API key."
- 402: "Insufficient credits."
- 429: "Too many requests. Please try again later."

## Step 5: Poll for Result

Show "Generating your portfolio... (usually 30s-2min)" then poll every 5 seconds.

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

## Step 6: Show Result

### On success

```
Your portfolio is ready!

View: {gammaUrl}
Download PDF: {exportUrl}

You can edit text, images, and layout in the Gamma editor.
```

### On failure

Analyze the error and ask if the user wants to retry.

### On timeout (over 5 min)

```
Generation ID: {generationId}
Check back later at https://gamma.app in your documents.
```

## Error Reference

| Error | Action |
|-------|--------|
| Key not set | Show setup guide |
| 401 | Ask to verify API key |
| 402 | Direct to Gamma dashboard to add credits |
| 429 | Wait and retry |
| Resume too short | Ask for more detail |
| Generation failed | Analyze error + offer retry |

## Notes

- Requires Gamma Pro, Ultra, Teams, or Business plan
- Each generation consumes Gamma credits
- API key: https://gamma.app > Account Settings > API Keys
- Generated portfolios are editable in Gamma's editor
- Export URLs expire after ~7 days
