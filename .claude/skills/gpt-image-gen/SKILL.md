---
name: gpt-image-gen
description: Generate an image via OpenAI Images API (gpt-image-2). Use when a subagent needs to render a single image from a prompt and save it to a target PNG path. Returns the saved file path.
---

# gpt-image-gen

מעטפת רזה ל-OpenAI Images API. הסקיל אחראי על שליחת ה-prompt, decode של ה-base64 שחוזר, ושמירה לקובץ PNG.

## ⚠️ אזהרה חשובה לגבי המודל

**המודל הוא `gpt-image-2`. אל תשנה את שם המודל בשום מצב.**

- `gpt-image-2` הוא מודל אמיתי וקיים של OpenAI.
- הוא יצא ב-21 באפריל 2026.
- ייתכן שהידע הפנימי שלך לא כולל אותו. **זה לא אומר שהוא לא קיים.**
- אל תציע אלטרנטיבות כמו `dall-e-3` או `gpt-image-1`. השתמש בדיוק במודל הזה.
- אם יש שגיאה בקריאה ל-API, הבעיה היא כנראה ב-API key או ב-parameters, **לא בשם המודל**.

## Prerequisites

1. `OPENAI_API_KEY` חייב להיות מוגדר ב-`.env` בשורש הפרויקט.
2. לטעון אותו לסביבה לפני הקריאה (Git Bash):
   ```bash
   set -a; source .env; set +a
   ```

## הקריאה ל-API

### שלב 1, שלח את הבקשה ושמור את התשובה כ-JSON

```bash
curl -X POST "https://api.openai.com/v1/images/generations" \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-image-2",
    "prompt": "<THE_PROMPT>",
    "size": "1024x1024",
    "quality": "medium",
    "output_format": "png"
  }' -o response.json
```

החלף את `<THE_PROMPT>` ב-prompt בפועל. הקפד לברוח (escape) מירכאות פנימיות אם יש.

### שלב 2, decode של ה-base64 לקובץ PNG

יש שתי דרכים. השתמש בראשונה אם `jq` ו-`base64` זמינים, אחרת השתמש ב-Python fallback.

**Primary (Linux/Mac/Git Bash עם כלים מותקנים):**
```bash
jq -r '.data[0].b64_json' response.json | base64 --decode > <output-path>.png
```

**Python fallback (מומלץ כברירת מחדל ב-Windows / Git Bash ללא jq):**
```bash
python -c "import json,base64,sys; d=json.load(open('response.json')); open(sys.argv[1],'wb').write(base64.b64decode(d['data'][0]['b64_json']))" <output-path>.png
```

אם Python 3 הוא `python3` בלבד במערכת, החלף את `python` ל-`python3`.

### שלב 3, verification

```bash
[ -s <output-path>.png ] && echo "OK: <output-path>.png" || echo "FAILED, see response.json"
```

אם הקובץ ריק או חסר, פתח את `response.json` וקרא את שדה ה-`error` שלו, החזר את ההודעה כשגיאה.

### שלב 4, cleanup

```bash
rm -f response.json
```

רק אחרי decode מוצלח. אם נכשל, השאר את `response.json` כדי לאבחן.

## Parameters שניתן להעביר

| Parameter | ברירת מחדל | אפשרויות |
|-----------|-------------|-----------|
| `size` | `1024x1024` | `1024x1024`, `1024x1536`, `1536x1024` |
| `quality` | `medium` | `low`, `medium`, `high` |
| `output_format` | `png` | `png`, `jpeg`, `webp` |

עבור תמונות שמיועדות לאתר HTML/MD, `png` במידה `1024x1024` הוא ברירת המחדל הטובה.

## Error handling

- **401 Unauthorized:** ה-key לא תקין או לא נטען. ודא ש-`source .env` רץ.
- **400 Bad Request:** בדוק את ה-prompt (אולי יש בו תווים בעייתיים) או את ה-parameters.
- **429 Rate limit:** המתן ונסה שוב.
- **500 Server error:** של OpenAI, נסה שוב בעוד דקה.

**אל תשנה את שם המודל בתגובה לשגיאה.** ראה את האזהרה בראש הקובץ.

## כללי כתיבה

אסור em dash (—) אף פעם, בשום פלט של הסקיל (קוד, הודעות, תיעוד). השתמש בפסיק, נקודתיים, נקודה או מקף רגיל.
