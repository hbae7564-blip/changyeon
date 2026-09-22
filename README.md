# changyeon — 창연명리 하네스 설정 백업

이 저장소는 창연명리 프로젝트를 운영하는 **Claude Code 하네스 설정**의 백업처입니다.
사이트나 챗봇의 실제 소스코드는 여기 없습니다 (아래 저장소 참고).

## 무엇이 들어있나

| 경로 | 설명 |
|------|------|
| `CLAUDE.md` | 하네스 전체 설명 + 변경 이력 |
| `.claude/agents/site-manager.md` | 사이트(index.html, 사진) 관리 에이전트 |
| `.claude/agents/bot-developer.md` | 카카오 챗봇 서버 개발 에이전트 |
| `.claude/agents/deploy-ops.md` | 배포·백업 운영 에이전트 |
| `.claude/agents/qa-reviewer.md` | 수정 직후 정합성을 검증하는 QA 에이전트 |
| `.claude/skills/changyeon-ops/` | 요청을 알맞은 전문가에게 배정하는 오케스트레이터 |
| `.claude/skills/site-content-update/` | 사이트 콘텐츠 수정 절차 |
| `.claude/skills/kakao-skill-server/` | 카카오 스킬 서버 수정 절차 |
| `.claude/skills/render-deploy-ops/` | Render 배포·백업 점검 절차 |
| `.claude/skills/changyeon-qa/` | 수정 직후 회귀 테스트·정합성 검증 절차 |

에이전트 4개, 스킬 5개 구성입니다.

## 관련 저장소

- 사이트: https://github.com/changyeon4277/changyeon4277.github.io
- 카카오 챗봇: https://github.com/hbae7564-blip/changyeon-kakao-bot

## 노트북이 바뀌거나 고장났을 때 복구 방법

바탕화면에 `claude` 폴더를 만들고, 그 안에서 아래 3개를 모두 내려받으면 됩니다.

```
git clone https://github.com/hbae7564-blip/changyeon.git
git clone https://github.com/changyeon4277/changyeon4277.github.io.git
git clone https://github.com/hbae7564-blip/changyeon-kakao-bot.git
```

그 다음 `changyeon` 폴더 안의 `.claude` 폴더와 `CLAUDE.md`를 바로 위(`claude` 폴더)로 복사하면
하네스가 원래대로 동작합니다.

## 주의

- API 키, 토큰, 비밀번호는 **절대 이 저장소에 넣지 않습니다.**
- 카카오봇의 실제 환경변수 값은 Render 대시보드와 로컬 `.env` 파일에만 두세요 (`.env`는 git에 올라가지 않도록 처리되어 있습니다).
