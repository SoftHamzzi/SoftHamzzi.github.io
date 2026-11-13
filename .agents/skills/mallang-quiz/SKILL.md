---
name: mallang-quiz
description: Generates a new 말랑말랑 퀴즈 (daily quiz) from this blog's posts using the MallangQuiz tool at C:\Github\MallangQuiz. Use when the user says "말랑 퀴즈 만들어줘", "/quiz", "퀴즈 생성", or asks for a new quiz. Does NOT grade; grading is the mallang-grade skill.
---

# Mallang Quiz (generate)

The generator lives in a separate repo: `C:\Github\MallangQuiz`. Its procedure is the source of truth; do not copy it here.

1. Read `C:\Github\MallangQuiz\AGENTS.md`, then `C:\Github\MallangQuiz\.claude\commands\quiz.md` in full. These are local procedure documents; Claude execution or quota is not required.
2. Follow every step in it. All `node scripts/...`, `mkdir`, and relative paths (`quiz-templates/`, `scripts/`, `mallang-quiz.config.json`, `.quiz-wrongnote/`) run with `cwd: C:\Github\MallangQuiz` (bash `cwd` parameter, not `cd`). Read/Glob of those relative paths must use the absolute `C:\Github\MallangQuiz\...` form.
3. Output goes to `C:\Github\MallangQuiz\quiz-output\{날짜시간}\quiz.md` and `answers.md`.
4. Do not copy the quiz into the blog unless the user asks. If asked, copy `quiz.md` to `_posts/routine/mallang_quiz/26/{M}/YYYY-MM-DD-yymmdd_mallang_quiz.md` with frontmatter matching the latest file in that folder (title `[말랑 퀴즈] yy/mm/dd 문제`, category `MallangQuiz`, tag `[Quiz]`) and the notice line the existing posts carry.
5. Answer to the user in Korean, per user preferences.
