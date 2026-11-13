---
name: retro-feedback
description: 일간·주간·월간·연간 5F 회고의 Feedback을 작성한다. '회고 피드백 써줘'와 지정한 회고 파일 경로에 사용한다. 말랑퀴즈 생성·채점에는 사용하지 않는다. 회고 전체를 대신 쓰거나 Future action·선택 학습을 바꾸지 않는다.
---

# 회고 피드백

1. 블로그 루트 `C:/Github/gitblog/SoftHamzzi.github.io`의 `AGENTS.md`와 `.claude/skills/retro-feedback/SKILL.md`를 읽고 원본 절차를 따른다. 이 파일들은 로컬 작업 지침이며 Claude 실행·로그인·할당량을 요구하지 않는다.
2. 대상 회고 파일을 지정받는다. 경로가 없으면 묻고, 사용자가 요청하지 않은 최신 파일을 임의로 골라 수정하지 않는다.
3. 원본의 기간별 과거 기록 읽기, 5F 표와 코멘트 형식, 기존 피드백 덮어쓰기 확인을 그대로 적용한다.
4. Fact·Feeling·Finding·Future action·이미 작성한 `🌙 남기는 말`은 보존한다. 선택 학습을 자동으로 고르지 않는다. frontmatter는 원본의 `last_modified_at` 예외만 따른다.
5. 한국어로 보고한다. 커밋·푸시·게시와 다른 글 수정은 별도 요청 없이는 하지 않는다.
