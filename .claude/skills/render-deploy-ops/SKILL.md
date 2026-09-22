---
name: render-deploy-ops
description: "changyeon-kakao-bot의 Render 배포, 환경변수(.env/SLACK_WEBHOOK_URL/KAKAO_* 키) 설정, GitHub Actions keep-alive 워크플로우, 그리고 3개 저장소(changyeon4277.github.io, changyeon-kakao-bot, changyeon)의 로컬-원격 동기화(백업) 상태 점검을 수행. '배포해줘', '서버가 안 돼요', '환경변수 설정', '백업 상태 확인', '노트북 바뀌어도 안전한지 점검' 등 배포·운영·백업 관련 요청에 반드시 사용. 사이트 콘텐츠 수정은 site-content-update, 챗봇 로직 수정은 kakao-skill-server를 사용."
---

# 배포·운영·백업 점검

이 프로젝트는 노트북 자체 백업이 되지 않는 환경이므로, **각 저장소가 GitHub 원격과 항상 동기화되어 있는 상태**가 실질적인 백업선이다. 모든 점검은 이 전제를 기준으로 한다.

## 저장소 동기화(백업) 점검 절차

3개 저장소(`changyeon4277.github.io`, `changyeon-kakao-bot`, `changyeon`) 각각에 대해:

```bash
git status
git fetch origin
git log HEAD..origin/main --oneline   # 원격에만 있는 커밋 (로컬이 뒤처짐)
git log origin/main..HEAD --oneline   # 로컬에만 있는 커밋 (아직 안 올라간 백업 공백)
```

- `git status`에 커밋되지 않은 변경(Untracked/Modified)이 있으면 **가장 먼저** 커밋 여부를 사용자에게 확인한다 — 노트북에 문제가 생기면 이 부분이 그대로 유실된다
- 로컬에만 있는 커밋이 있으면 즉시 `git push`를 제안한다
- 정기 점검(예: 작업 세션 종료 시)마다 이 3단계를 습관적으로 실행하는 것을 사용자에게도 권장한다

## Render 배포 설정 (changyeon-kakao-bot)

- Build Command: `npm install`
- Start Command: `npm start`
- Environment 변수: `.env.example`을 기준으로 `SLACK_WEBHOOK_URL`, `KAKAO_REST_API_KEY`, `KAKAO_CLIENT_SECRET`, `KAKAO_REFRESH_TOKEN` 등을 Render 대시보드의 Environment 탭에 등록 — **Claude가 실제 키 값을 대신 입력하거나 값을 요청하지 않는다.** 어디에 입력하는지 화면 경로만 안내한다
- 배포 후 `https://xxxx.onrender.com`에 접속해 "정상적으로 실행 중입니다"가 뜨는지 확인

## keep-alive 워크플로우

`.github/workflows/keep-alive.yml`은 Render 무료 플랜의 슬립(15분 미접속 시 잠자기)을 방지하기 위해 주기적으로 서버에 핑을 보내는 GitHub Actions 워크플로우다. 점검 시:
- 워크플로우의 대상 URL이 현재 실제 Render 서비스 URL과 일치하는지 확인 (서비스를 재생성하면 URL이 바뀔 수 있음)
- GitHub 저장소의 Actions 탭에서 최근 실행이 성공(초록색)인지 확인하라고 안내

## 장애 대응 순서

"챗봇이 안 돼요" / "사이트가 안 떠요" 보고를 받으면:
1. 어느 쪽 문제인지 먼저 구분 (사이트 vs 챗봇) — 구분이 안 되면 둘 다 순서대로 확인
2. Render 서비스가 실행 중인지, 최근 배포가 실패하지 않았는지 확인
3. 최근 커밋이 배포에 반영됐는지(Render는 push 시 자동 재배포) 확인
4. 문제가 코드 로직이면 `kakao-skill-server`로, 콘텐츠면 `site-content-update`로 위임

## changyeon 저장소 — 하네스 설정 백업 절차

`changyeon` 저장소는 이 워크스페이스 루트(`C:\Users\wuenw\Desktop\claude`)의 `.claude/`(agents/skills)와 `CLAUDE.md`를 백업하는 용도로 확정됐다. 이 두 파일/폴더는 다른 3개 git 저장소 어디에도 속하지 않으므로, 수정한 뒤 그대로 두면 노트북 문제 시 유실된다.

**"하네스 설정 백업해줘" 요청을 받으면:**
1. 루트의 `.claude/agents/`, `.claude/skills/`, `CLAUDE.md`를 `changyeon/` 폴더 안 동일 경로에 덮어쓰기 복사 (원본 3개 저장소 내부 파일은 절대 건드리지 않는다)
2. `changyeon` 폴더에서 `git status`로 실제 변경분 확인
3. 무엇이 바뀌었는지 알아볼 수 있는 커밋 메시지로 커밋 (예: "QA 에이전트 추가 반영")
4. `git push`
5. `changyeon/README.md`의 설명이 최신 구성과 어긋나면 함께 갱신 (에이전트/스킬 개수, 복구 절차)

**자동 동기화가 아님을 항상 인지한다**: 루트 설정을 고친 직후 자동으로 `changyeon`에 반영되지 않는다. 코드/스킬을 수정하는 작업(에이전트 추가, 오케스트레이터 수정 등) 마지막 단계에 이 백업 절차를 습관적으로 제안한다.
