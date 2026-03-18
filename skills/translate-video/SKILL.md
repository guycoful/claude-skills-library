---
name: translate-video
description: "Translate video subtitles to any language with native-quality refinement. Full pipeline: transcribe → translate → refine → embed RTL-safe subtitles. Use for: translate video, תרגם סרטון, video translation, foreign subtitles, Hebrew subtitles, translated captions."
argument-hint: "[video-path] [target-language] [--shorts|--regular]"
---

# Translate Video

End-to-end video translation pipeline. Takes a video, transcribes it, translates to target language, and burns subtitles onto the video.

## Usage

```
/translate-video /path/to/video.mp4 he
/translate-video /path/to/video.mp4 he --shorts
/translate-video /path/to/video.mp4 he --regular
```

**Arguments:**
- `$1` - Path to video file (required)
- `$2` - Target language code (default: `he`). Codes: he, ar, es, fr, de, ru, zh, ja, etc.
- `$3` - Mode flag (default: `--regular`):
  - `--shorts` — Short subtitles, preserve original timestamps, good for Reels/TikTok/Stories
  - `--regular` — Longer subtitles, merge entries into readable sentences, good for YouTube/tutorials

**If no mode is specified, ask the user:** "What type of video is this — Shorts/Stories or a regular video?"

---

## Pipeline

### Step 1: Transcribe (via `/transcribe` skill)

Extract audio if video is too large (>25MB audio), then transcribe:

```bash
# Extract audio if needed (reduces upload size)
ffmpeg -i "$VIDEO" -vn -acodec libmp3lame -ab 128k "$AUDIO_PATH" -y

# Transcribe using the transcribe skill's script — always include --json to get word-level timestamps
cd ~/.claude/skills/transcribe/scripts && [ -d node_modules ] || npm install --silent
npx ts-node transcribe.ts -i "$INPUT" -o "$SRT_PATH" --json
```

This generates:
- `{basename}.srt` - Raw SRT file (chunked, for reference)
- `{basename}.md` - Readable text
- `{basename}_transcript.json` - **Word-level timestamps** (use this for precise merging)

**The JSON contains every word with its exact `start` and `end` in seconds:**
```json
{
  "words": [
    { "word": "All", "start": 0.329, "end": 0.511 },
    { "word": "right,", "start": 0.511, "end": 0.729 },
    ...
  ]
}
```

---

### Step 2: Translate

Read the `.md` file to understand full context, then translate the `.srt` file.

**Translation rules (both modes):**
- Translate ALL subtitle text entries, preserving SRT index numbers and timestamps exactly
- Do NOT translate proper nouns (product names, people's names, brand names)
- Keep technical terms commonly used untranslated (API, CLI, SaaS, etc.)
- Adapt idioms to natural equivalents in the target language
- Match the speaker's register (casual/formal)

---

### Step 3: Refinement — TWO MODES

#### MODE A: `--shorts` (Preserve Timestamps)

**PRIMARY RULE: DO NOT change, merge, or regroup timestamps. Each SRT entry keeps its original timestamps exactly.**

Fix only the text within each entry:
1. **Fix grammar and naturalness** in the target language — minimal edits only
2. **Do NOT merge entries** — keep the same number of entries as the translation
3. **Do NOT move words between entries** — if a phrase is split mid-sentence, leave it
4. **Only rephrase within each entry** to sound more natural, no added/removed content

**Line limits:**
- **MAX 2 lines per entry**
- **MAX 38 chars per line**
- If you MUST split an entry (only when unavoidable): divide timestamp proportionally

**What NOT to do:**
- ❌ Do NOT merge 2+ entries into 1
- ❌ Do NOT regroup words across entry boundaries
- ❌ Do NOT change timestamps unless splitting (and even then, split proportionally)

---

#### MODE B: `--regular` (Merge into Readable Sentences)

**Goal: readable, comfortable subtitles for regular video viewing.**

**Uses word-level timestamps for precision** — do NOT use SRT entry boundaries for timing.

1. **Read `.md`** to understand full context and meaning — do NOT read the full JSON
2. **Decide subtitle groupings** — group the original words into natural sentences/clauses
3. **For each subtitle boundary**, look up the specific word in `_transcript.json` to get its exact timestamp — only fetch the words you need (first and last word of each group)
4. **Timestamps:** subtitle start = first word's `start`, subtitle end = last word's `end` — precise to the millisecond
5. **Target 1 full sentence per subtitle entry** (or one natural clause if sentence is long)
6. **Native speaker test** — each subtitle should read as a complete, natural thought

**Python helper to find a specific word's timestamp:**
```python
import json
with open('PATH_TO_transcript.json') as f:
    words = json.load(f)['words']

# Find timestamp of a specific word by index or text
# words[0]  → first word: {word, start, end}
# words[-1] → last word
# Search: next(w for w in words if w['word'].strip('.,') == 'everyone')
```

**Line limits:**
- **MAX 2 lines per entry**
- **MAX 42 chars per line** (slightly wider than shorts, still safe for 16:9 video)
- Split long lines at natural word boundaries

**Target entry count:** aim for roughly 1 entry per 3–5 seconds of speech (much fewer than raw)

**What's allowed in Regular mode (unlike Shorts):**
- ✅ Merge multiple raw entries into one
- ✅ Move words across entry boundaries for natural phrasing
- ✅ Adjust timestamps to match merged content
- ✅ Rephrase freely for naturalness within the same meaning

**Punctuation preference (Regular mode):**
- When merging entries, **prefer to break at points where punctuation already exists** (period, comma, etc.)
- This means subtitles will naturally tend to end with `.` or `,` — but only because you're cutting there, not because you added anything
- Do NOT add punctuation that wasn't in the original translation

**What's NOT allowed even in Regular mode:**
- ❌ Add content that wasn't said
- ❌ Remove content that was said
- ❌ Change meaning
- ❌ Add punctuation marks that don't exist in the translation

---

### Step 3 Post-processor (run after EITHER mode)

Replace `INPUT` and `MAX_CHARS` based on mode (38 for --shorts, 42 for --regular):

```python
python3 << 'EOF'
import re

INPUT = 'PATH_TO_TRANSLATED_SRT'
MAX_CHARS = 42  # use 38 for --shorts, 42 for --regular

def ts_to_ms(ts):
    h,m,rest = ts.split(':'); s,ms = rest.split(',')
    return int(h)*3600000+int(m)*60000+int(s)*1000+int(ms)

def ms_to_ts(ms):
    h=ms//3600000; ms%=3600000; m=ms//60000; ms%=60000; s=ms//1000; ms%=1000
    return f'{h:02d}:{m:02d}:{s:02d},{ms:03d}'

def split_line(line, max_chars):
    clean = line.strip('\u202b\u202c\u200f\u200e')
    if len(clean) <= max_chars: return [clean]
    words = clean.split(' '); lines = []; current = ''
    for word in words:
        test = (current+' '+word).strip()
        if len(test) <= max_chars: current = test
        else:
            if current: lines.append(current)
            current = word
    if current: lines.append(current)
    return lines

with open(INPUT, 'r', encoding='utf-8') as f: content = f.read()
blocks = re.split(r'\n\n+', content.strip())
result_entries = []
for block in blocks:
    lines = block.split('\n')
    if len(lines) < 3: continue
    ts_match = re.match(r'(\d{2}:\d{2}:\d{2},\d{3}) --> (\d{2}:\d{2}:\d{2},\d{3})', lines[1])
    if not ts_match: continue
    start_ts, end_ts = ts_match.group(1), ts_match.group(2)
    text_lines = [l.strip('\u202b\u202c\u200f\u200e') for l in lines[2:] if l.strip()]
    all_sublines = []
    for tl in text_lines: all_sublines.extend(split_line(tl, MAX_CHARS))
    chunks = [all_sublines[i:i+2] for i in range(0, len(all_sublines), 2)]
    start_ms = ts_to_ms(start_ts); end_ms = ts_to_ms(end_ts)
    chunk_ms = max(1, (end_ms-start_ms)//len(chunks))
    for i, chunk in enumerate(chunks):
        c_start = ms_to_ts(start_ms+i*chunk_ms)
        c_end = ms_to_ts(start_ms+(i+1)*chunk_ms-1) if i<len(chunks)-1 else end_ts
        result_entries.append((c_start, c_end, chunk))
output_lines = []
for idx, (start, end, tls) in enumerate(result_entries, 1):
    output_lines += [str(idx), f'{start} --> {end}'] + ['\u202b'+l+'\u202c' for l in tls] + ['']
with open(INPUT, 'w', encoding='utf-8') as f: f.write('\n'.join(output_lines))
print(f"Done: {len(result_entries)} entries, max {max(len(l.strip(chr(0x202b)+chr(0x202c))) for _,_,ls in result_entries for l in ls)} chars/line")
EOF
```

---

### Step 3.5: RTL Fix (MANDATORY for Hebrew/Arabic/Farsi)

**Apply IMMEDIATELY after writing the SRT — before embed.**

```python
python3 -c "
import re

with open('SRT_FILE_PATH', 'r', encoding='utf-8') as f:
    content = f.read()

lines = content.split('\n')
result = []
for line in lines:
    stripped = line.strip()
    if stripped and not re.match(r'^\d+$', stripped) and not re.match(r'\d{2}:\d{2}:\d{2}', stripped):
        clean = line.strip('\u202b\u202c\u200f\u200e')
        line = '\u200F\u202B' + clean
    result.append(line)

with open('SRT_FILE_PATH', 'w', encoding='utf-8') as f:
    f.write('\n'.join(result))
"
```

- **U+200F** (RLM) + **U+202B** (RLE) — required for libass+FriBidi RTL rendering
- Do NOT add U+202C at end — interferes with RTL detection

---

### Step 4: Embed Subtitles

**Preferred:** Use `~/.local/bin/ffmpeg-ass` (custom-compiled with libass, native ARM).

```bash
~/.local/bin/ffmpeg-ass -i "$VIDEO" \
  -vf "subtitles=$TRANSLATED_SRT:force_style='FontSize=24,PrimaryColour=&H00FFFFFF,OutlineColour=&H00000000,Outline=2,Shadow=1,Alignment=2,MarginV=30'" \
  -c:v libx264 -preset fast -crf 23 -c:a copy "$OUTPUT" -y
```

**WARNING:** Do NOT use Docker ffmpeg (`jrottenberg/ffmpeg:5-alpine`) for long videos on ARM Mac — x86 emulation is ~100x slower and hangs on videos >1min.

### Step 5: Open Result

```bash
open "$OUTPUT"  # macOS
```

---

## Output Files

All files saved next to the original video:

| File | Description |
|------|-------------|
| `{name}.srt` | Original language SRT |
| `{name}.md` | Original readable transcript |
| `{name}_{lang}.srt` | Translated + refined SRT |
| `{name}_{lang}_subtitled.mp4` | Final video with burned-in subtitles |

---

## Mode Summary

| | `--shorts` | `--regular` |
|---|---|---|
| **Merge entries?** | ❌ Never | ✅ Yes, into sentences |
| **Change timestamps?** | ❌ Never | ✅ Yes, to match merged entries |
| **Max chars/line** | 38 | 42 |
| **Best for** | Reels, TikTok, Stories | YouTube, tutorials, talks |
| **Entry count** | Same as raw | ~3–5x fewer |

---

## Language Codes

| Code | Language | RTL? |
|------|----------|------|
| `he` | Hebrew | Yes |
| `ar` | Arabic | Yes |
| `fa` | Farsi | Yes |
| `en` | English | No |
| `es` | Spanish | No |
| `fr` | French | No |
| `de` | German | No |
| `ru` | Russian | No |
| `zh` | Chinese | No |
| `ja` | Japanese | No |
| `pt` | Portuguese | No |
| `it` | Italian | No |

## Examples

```
/translate-video ~/Desktop/tutorial.mp4 he --regular
/translate-video ~/Downloads/reel.mp4 he --shorts
/translate-video ~/Desktop/talk.mp4 es --regular
```
