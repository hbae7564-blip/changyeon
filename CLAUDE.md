## 하네스: 창연명리 (changyeon)

**목표:** 사이트(changyeon4277.github.io) + 카카오챗봇(changyeon-kakao-bot) 운영을 전문 에이전트 팀으로 지원하고, 노트북 백업이 안 되는 환경에서 GitHub 원격 동기화를 실질적 백업선으로 관리한다. `changyeon` 저장소는 이 워크스페이스 루트의 `.claude/`와 `CLAUDE.md`(하네스 설정 자체)를 백업하는 용도로 확정.

**트리거:** 이 폴더 하위 프로젝트(사이트/챗봇/배포/백업) 관련 작업 요청 시 `changyeon-ops` 스킬을 사용하라. 단순 질문은 직접 응답 가능.

**변경 이력:**
| 날짜 | 변경 내용 | 대상 | 사유 |
|------|----------|------|------|
| 2026-09-22 | 초기 구성 (site-manager, bot-developer, deploy-ops 3개 에이전트 + changyeon-ops 오케스트레이터) | 전체 | 노트북 백업 불가 → 프로젝트를 이 폴더로 이전, 하네스로 운영 체계 구축 |
| 2026-09-22 | 전체 백업 실행 (3개 저장소 동기화 확인 + changyeon 저장소를 하네스 설정 백업소로 확정) | changyeon 저장소 | 사용자 요청 "백업해줘 전부" |
| 2026-09-22 | qa-reviewer 에이전트 + changyeon-qa 스킬 추가, changyeon-ops에 점진적 QA 단계 삽입 | agents/qa-reviewer.md, skills/changyeon-qa, skills/changyeon-ops | 사용자 요청 "하네스 엔지니어링" — QA/검증 에이전트 추가 |
