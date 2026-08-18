---
name: retro-feedback
description: Writes the "🔁 Feedback(피드백)" section of a 5F-format retrospective (회고) post in this blog's _posts/routine/{daily,weekly,monthly,yearly} tree, given a file path. Use this whenever the user says something like "이 회고에 피드백 써줘", "회고 피드백 채워줘", pastes a path under _posts/routine, or otherwise asks Claude to fill in only the Feedback part of a daily/weekly/monthly/yearly retrospective while Fact/Feeling/Finding/Future action are already written. Do NOT use this for writing an entire retrospective from scratch, and do not touch Fact/Feeling/Finding/Future action or the closing "🌙 남기는 말" — only the Feedback section is this skill's job.
---

# Retro Feedback

Fill in the missing `## 🔁 Feedback(피드백)` section of one retrospective post. The user always writes Fact/Feeling/Finding/Future action (and usually the closing 🌙 남기는 말) themselves; this skill plays the role of a coach reading that entry against the person's actual history and responding honestly — not the role of a cheerleader summarizing what was already said.

## Input

The user gives (or should be asked for) an absolute or repo-relative path to a single markdown file under `_posts/routine/{daily,weekly,monthly,yearly}/...`. If no path is given, ask for one rather than guessing — do not scan the directory for "the most recent file needing feedback" unless the user explicitly asks for that.

## Step 1 — Read and validate the target file

Read the file. Confirm:
- Front matter has a `[Daily]`/`[Weekly]`/`[Monthly]`/`[Yearly]` tag, or the path tells you the cadence (`daily/`, `weekly/`, `monthly/`, `yearly/`).
- `## 🧩 Fact`, `## 💭 Feeling`, `## 💡 Finding`, `## 🎯 Future action` are already filled with real content.
- `## 🔁 Feedback(피드백)` is missing, or present with only the boilerplate quote line (`> 앞서 정한 향후 행동을 실천해본 뒤...`) and no body under it.

If Fact/Feeling/Finding/Future action look empty or clearly unfinished, stop and tell the user — writing Feedback against an incomplete entry produces generic, useless output. If a Feedback body already exists with real content, confirm with the user before overwriting it.

## Step 2 — Gather context by cadence

The whole value of this section is that it's *specific* — it cites real dates, streaks, and whether past advice was actually followed. Generic encouragement ("잘하고 있어요!") is a failure mode. To avoid it, read backward before writing anything:

- **daily**: read the 5–7 most recent prior daily entries (same `daily/{yy}/{m}/` folder and the one before it, sorted by date). You're looking for: recurring goals/routines (GAS, 모던 C++, 퀴즈, 선형대수 etc. — whatever *this* person's routine items are, don't assume), streaks or gaps in writing itself, and — most important — what the *previous* entry's Feedback section told them to do next, so you can check whether it happened.
- **weekly**: read every daily entry that falls inside the week being reviewed, plus the 1–2 most recent prior weekly entries for trend continuity.
- **monthly**: read every weekly entry inside that month, plus the 1–2 most recent prior monthly entries.
- **yearly**: read every monthly entry in that year, plus the prior yearly entry if one exists.

If sibling files are sparse (e.g. early in the blog's history, or a cadence that's new), work with what exists — don't fabricate history that isn't there.

## Step 3 — Find the actual thread, not just a summary

Before writing, identify in your own head:
1. What did this person commit to last time (previous entry's "다음 단계" / Future action), and did today's Fact section show it happening, half-happening, or not at all?
2. Is there a pattern across the last several entries — a recurring avoidance (e.g. always finishing the "safe" task and dropping the "hard" one), a streak worth naming, an external disruption worth separating from a real behavior problem?
3. What's the one thing worth being direct about? Not every point — pick what actually matters this time.

This is what makes the Feedback section read like it was written by someone who's been paying attention across entries, not someone reacting only to today's text.

## Step 4 — Write the section, matching the cadence's structure exactly

All cadences share a table with columns `5F 단계 | 주제 | 내용`, using rows `S(상황)`, `B(현재 계획)`, `I(영향)`, `N(다음 단계)`, `F(후속 조치)` (row spans are simulated by leaving the first cell blank on continuation rows, as markdown tables do here — see examples below). Within `I` and `N`, mark items `✅` (real win, say so plainly), `⚠️` (a problem, named specifically with the evidence), or `💡` (an insight worth flagging). Bold the key phrase in each cell's "주제" column.

Read one or two recent same-cadence files in this repo before writing (e.g. a recent `daily/*.md`, `weekly/*.md`, `monthly/*.md`, or the single `yearly/*.md`) to calibrate exact tone and table density — the structure below is the skeleton, the real files are the style reference.

### daily
```
## 🔁 Feedback(피드백)
> 앞서 정한 향후 행동을 실천해본 뒤, 이에 대해 어떤 피드백을 받았나?

| 5F 단계 | 주제 | 내용 |
|--------|------|------|
| **S** (상황) | <날짜/요일 — 한줄 상황> | <Fact를 압축한 사실관계> |
| **B** (현재 계획) | <이번 주/기간 잔여 계획> | <...> |
| **I** (영향) | ✅ <구체적 성과> | <근거> |
| | ⚠️ <구체적 문제> | <근거, 가능하면 날짜 인용> |
| | 💡 <인사이트> | <...> |
| **N** (다음 단계) | 1. | <내일 가장 먼저 할 것> |
| | 2. | <...> |
| **F** (후속 조치) | 내일 (<날짜>) | <다음 회고에서 확인할 체크리스트> |

---

**<한 문장 헤드라인 — 오늘의 핵심을 요약>**

<2~4문장의 직접적인 코멘트. 과거 회고의 구체적 날짜/발언을 인용해서 패턴을 짚고, 내일 지켜볼 것 하나를 명시.>
```
No `## 💬 한 줄 요약` or `## 💡 코멘트` heading in daily — the bold headline + short paragraph right after the table *is* the comment, ending the section (the file's own `# 🌙 남기는 말` follows after, already written by the user — leave it untouched).

### weekly / monthly
Same table shape as daily but rows summarize the week/month (S references sub-periods like "[1주차]/[2주차]" for monthly). After the table, add two more subsections before the closing 🌙:
```
---

## 💬 한 줄 요약

**<기간 요약 한 문장, 굵게>**
**<다음 기간 목표 한 문장, 굵게>**

---

## 💡 코멘트

<2~4개의 짧은 문단. 성과는 있는 그대로 인정하고(✅), 그냥 격려로 끝내지 말고 다음 기간에 실제로 다르게 할 것 하나를 짚는다.>

---
```

### yearly
Same table + `💬 한 줄 요약`, but replace `💡 코멘트` with a more confrontational `🔥 직언` section — this is the one place the coach voice goes further than validation:
```
---

## 🔥 직언 (<이 회고에서 다룰 핵심 질문/발언>)

> "<Fact나 Future action에서 인용한 본인의 말 — 안일한 계획이나 회피성 발언일수록 좋은 인용 대상>"

**<정면으로 반박하는 한 문장>**

<과거 몇 개월/년의 구체적 증거를 들어 왜 지금 계획이 위험한지 설명. 숫자·날짜로 뒷받침.>

**구체적 질문 N개:**
1. <답을 피할 수 없는 구체적 질문>
2. ...

**해야 할 것:**
- <월별/기간별로 마감이 박힌 구체적 대안 계획>

---
```

## Step 5 — Insert into the file

Use Edit, not a full rewrite. Insert the Feedback body:
- If `## 🔁 Feedback(피드백)` with only the quote line already exists, insert your content directly after that quote line (before whatever comes next — a `---` and `# 🌙 남기는 말`, or end of file).
- If the whole `## 🔁 Feedback(피드백)` heading is missing, add it (with the quote line) in its correct position — after `## 🎯 Future action` and before `# 🌙 남기는 말` (or at the end of the file if there's no closing section yet).

Never modify Fact/Feeling/Finding/Future action or an already-written 🌙 남기는 말. If `date` is in the past relative to today (a catch-up entry written late) and `last_modified_at` in the front matter still equals the original `date`, update `last_modified_at` to today; otherwise leave front matter untouched.

## Step 6 — Report back

After editing, tell the user in 2-3 sentences what the Feedback centers on (the one thread you picked in Step 3) — not a restatement of the table. This gives them a fast way to say "no, that's not actually the issue" before treating it as final.
