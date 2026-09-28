---
name: retro-feedback
description: Writes the "🔁 Feedback(피드백)" section of a 5F-format retrospective (회고) post in this blog's _posts/routine/{daily,weekly,monthly,yearly} tree, given a file path, and — for daily entries that use the 필수/선택 루틴 split — also picks one item from the "⚡ 선택 학습 풀" to fill in the blank "오늘의 선택된 추가 학습:" line under Future action. Use this whenever the user says something like "이 회고에 피드백 써줘", "회고 피드백 채워줘", pastes a path under _posts/routine, or otherwise asks Claude to fill in the Feedback part of a daily/weekly/monthly/yearly retrospective while Fact/Feeling/Finding/Future action are already written. Do NOT use this for writing an entire retrospective from scratch, and never touch Fact/Feeling/Finding or the closing "🌙 남기는 말" — Feedback, plus that one blank line in Future action when present, are this skill's only job.
---

# Retro Feedback

Fill in the missing `## 🔁 Feedback(피드백)` section of one retrospective post, and — for daily entries only — the blank "오늘의 선택된 추가 학습:" line under `## 🎯 Future action`. The user always writes Fact/Feeling/Finding/Future action (and usually the closing 🌙 남기는 말) themselves; this skill plays the role of a coach reading that entry against the person's actual history and responding honestly — not the role of a cheerleader summarizing what was already said. Choosing tomorrow's extra study item is the same coaching judgment applied forward instead of backward: pick based on what the person actually needs next, not a round-robin.

## Input

The user gives (or should be asked for) an absolute or repo-relative path to a single markdown file under `_posts/routine/{daily,weekly,monthly,yearly}/...`. If no path is given, ask for one rather than guessing — do not scan the directory for "the most recent file needing feedback" unless the user explicitly asks for that.

## Step 1 — Read and validate the target file

Read the file. Confirm:
- Front matter has a `[Daily]`/`[Weekly]`/`[Monthly]`/`[Yearly]` tag, or the path tells you the cadence (`daily/`, `weekly/`, `monthly/`, `yearly/`).
- `## 🧩 Fact`, `## 💭 Feeling`, `## 💡 Finding`, `## 🎯 Future action` are already filled with real content.
- `## 🔁 Feedback(피드백)` is missing, or present with only the boilerplate quote line (`> 앞서 정한 향후 행동을 실천해본 뒤...`) and no body under it.
- (daily only) Whether `## 🎯 Future action` contains a `### ⚡ 선택 학습 풀` (or equivalently named) list and a line like `오늘의 선택된 추가 학습:` left blank after the colon. Not every daily entry uses this 필수/선택 split — only act on it when both the pool list and the blank line are actually present.

If Fact/Feeling/Finding/Future action look empty or clearly unfinished, stop and tell the user — writing Feedback against an incomplete entry produces generic, useless output. If a Feedback body already exists with real content, confirm with the user before overwriting it. Same for the selected-study line: if it's already filled in, leave it and don't ask to redo it unless the user says so.

## Step 2 — Gather context by cadence

The whole value of this section is that it's *specific* — it cites real dates, streaks, and whether past advice was actually followed. Generic encouragement ("잘하고 있어요!") is a failure mode. To avoid it, read backward before writing anything:

- **daily**: read the 5–7 most recent prior daily entries (same `daily/{yy}/{m}/` folder and the one before it, sorted by date). You're looking for: recurring goals/routines (GAS, 모던 C++, 퀴즈, 선형대수 etc. — whatever *this* person's routine items are, don't assume), streaks or gaps in writing itself, and — most important — what the *previous* entry's Feedback section told them to do next, so you can check whether it happened. If this entry has the 필수/선택 루틴 split, also track, across those same prior entries: which item was picked as "오늘의 선택된 추가 학습" each day, how long it's been since each pool item was last picked (or never), and what today's Fact section says actually got done (a pool item finished unprompted, a book chapter finished that opens up the next one, a stated intention like "모던 C++까지는 현재 방식 유지, 이후 책부터는...").
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

## Step 4 — (daily only) Pick today's "오늘의 선택된 추가 학습"

Skip this step entirely for weekly/monthly/yearly, and for daily entries that don't have the 선택 학습 풀 structure (Step 1 already checked this).

This pick becomes tomorrow's required extra study item, so treat it with the same seriousness as the Feedback table — a lazy round-robin defeats the point of the routine as the person described it (선택 루틴 중 하나가 다음 날의 필수 루틴이 되어 성취감을 준다).

1. List the current 선택 학습 풀 exactly as written in this entry's Future action section — items and any sub-options (e.g. "CS 공부 추가 학습 : 이펙티브 모던 C++ or 게임 프로그래밍 패턴") are per-entry, don't assume they match older entries.
2. From the history gathered in Step 2, note per pool item: last time it was picked, and whether it was actually followed through on afterward (per Fact sections of later entries) — a pattern of picking something and then not doing it is itself worth weighting against picking it again immediately.
3. Weigh, in this order: (a) something explicitly time-sensitive or blocking progress right now (e.g. a course/book chapter mentioned in today's Fact/Finding as needing follow-up), (b) an item that's gone unusually long without being picked relative to the others, (c) variety — avoid picking the same category two days running unless (a) clearly overrides it.
4. If the chosen item has sub-options (an "A or B" style entry), resolve it to the single concrete option that fits where the person actually is right now (e.g. continue the book already in progress rather than switching), based on Fact/Finding evidence, not a coin flip.
5. Write the one resolved choice as plain text after the colon on the blank line (e.g. `오늘의 선택된 추가 학습: 말랑 퀴즈`) — the specific pool item's wording, not a restatement of the whole pool.

## Step 5 — Write the section, matching the cadence's structure exactly

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

## Step 6 — Insert into the file

Use Edit, not a full rewrite. Two separate edits, both scoped as narrowly as possible:
- **Feedback body**: if `## 🔁 Feedback(피드백)` with only the quote line already exists, insert your content directly after that quote line (before whatever comes next — a `---` and `# 🌙 남기는 말`, or end of file). If the whole `## 🔁 Feedback(피드백)` heading is missing, add it (with the quote line) in its correct position — after `## 🎯 Future action` and before `# 🌙 남기는 말` (or at the end of the file if there's no closing section yet).
- **Selected-study line** (daily only, when Step 4 ran): replace only the blank after the colon on the `오늘의 선택된 추가 학습:` line with the chosen item's text. Leave every other line of Future action — the 필수 items, the pool list itself — exactly as written.

Never modify Fact/Feeling/Finding, the rest of Future action, or an already-written 🌙 남기는 말. If `date` is in the past relative to today (a catch-up entry written late) and `last_modified_at` in the front matter still equals the original `date`, update `last_modified_at` to today; otherwise leave front matter untouched.

## Step 7 — Report back

After editing, tell the user in 2-3 sentences what the Feedback centers on (the one thread you picked in Step 3) — not a restatement of the table. If Step 4 ran, add one line naming what you picked for tomorrow's extra study and the one main reason (the overdue item, the time-sensitive one, whichever won) — this gives them a fast way to say "no, that's not actually the issue" or "pick something else" before treating either as final.
