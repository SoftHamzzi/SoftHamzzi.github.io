---
name: mallang-grade
description: Grades a solved 말랑말랑 퀴즈 post in this blog (_posts/routine/mallang_quiz/...) and writes the *_mallang_answer.md next to it, then updates the wrong-note store in C:\Github\MallangQuiz. Use when the user says "말랑 퀴즈 채점해줘", "/grade", or gives a *_mallang_quiz.md path/filename to grade.
---

# Mallang Grade

Grading logic lives in the MallangQuiz repo (`C:\Github\MallangQuiz`). It is the source of truth; do not copy it here.

1. Need the quiz filename or path (e.g. `2026-11-02-261102_mallang_quiz.md`). If not given, ask; do not guess.
2. Read `C:\Github\MallangQuiz\AGENTS.md`, then `C:\Github\MallangQuiz\.claude\commands\grade.md` in full. Treat `$ARGUMENTS` there as the filename/path from step 1. These are local procedure documents; Claude execution or quota is not required.
3. Follow every step. All `node scripts/...`, `rm`, and relative paths (`quiz-templates/`, `.quiz-wrongnote/`) run with `cwd: C:\Github\MallangQuiz`; Read/Glob must use the absolute `C:\Github\MallangQuiz\...` form.
4. The quiz file (`*_quiz.md`) is read-only. Only the new `*_answer.md` (same folder) is written, plus the wrong-note update from step 5 of that doc.
5. If `answerExists` is true, confirm with the user before overwriting.
6. Report in Korean: total score, grade, per-question one-liner, wrong-note summary.
