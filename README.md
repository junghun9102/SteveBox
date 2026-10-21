# SteveBox

보고서를 깔끔한 웹 문서로 만들어 GitHub Pages로 공유하는 저장소입니다.

**리포트 목록:** https://junghun9102.github.io/SteveBox/report/

## 사용법

Claude Code 세션에서 "리포트 만들어줘", "이거 리포트로 정리해줘"라고 요청하면 `report` 스킬이 진행합니다.

1. 필요한 내용을 질문합니다.
2. 채팅에 마크다운 초안을 보여주고 확인을 받습니다.
3. 확정되면 `report/YYYY-MM-DD-slug.html`을 만들고 목록(`report/index.html`)에 추가합니다.
4. "올려줘"라고 요청하면 커밋·push하고 `main`으로 가는 PR을 만듭니다. PR을 머지하면 GitHub Pages에 공개됩니다.

## 구조

| 경로 | 내용 |
|---|---|
| `report/` | 공개되는 리포트 HTML, 목록 페이지(`index.html`), 공용 스타일(`style.css`) |
| `.claude/skills/report/` | 리포트 작성 스킬 (`SKILL.md`, 템플릿, 다이어그램 가이드) |
| `index.html` | 사이트 루트에서 `report/`로 이동 |
| `.nojekyll` | GitHub Pages의 Jekyll 처리를 끔 |

## 주의

이 저장소와 리포트 페이지는 공개됩니다. 회사 내부 정보나 민감정보는 넣지 마세요.
