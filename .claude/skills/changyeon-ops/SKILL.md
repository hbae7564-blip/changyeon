---
name: changyeon-ops
description: "창연명리 프로젝트(사이트 changyeon4277.github.io + 카카오챗봇 changyeon-kakao-bot + 하네스 설정 백업소 changyeon)의 요청을 적절한 전문 에이전트에게 배정하는 오케스트레이터. '사진 올려줘', '사이트 수정', '챗봇 고쳐줘', '오픈빌더 오류', '배포해줘', '서버 안 돼요', '백업 확인', '하네스 설정 백업해줘', '노트북 바뀌어도 괜찮은지', '검증해줘', 'QA' 등 이 프로젝트와 관련된 모든 요청, 그리고 '저번에 한 거 다시', '~만 다시 고쳐줘', '이어서 해줘' 같은 후속 요청 시 반드시 이 스킬을 먼저 사용해 어떤 전문가에게 맡길지 판단한다."
---

# Changyeon Ops — 창연명리 프로젝트 오케스트레이터

## 실행 모드: 하이브리드 (기본은 전문가 풀 / 서브 에이전트, 교차 영역 요청 시 에이전트 팀)

| 상황 | 모드 | 이유 |
|------|------|------|
| 한 영역(사이트 or 챗봇 or 배포)만 다루는 요청 | 서브 에이전트 (전문가 풀) | 단일 전문가 호출로 충분, 팀 통신 오버헤드 불필요 |
| 2개 이상 영역이 얽힌 요청 (예: "새 기능 만들고 배포까지") | 에이전트 팀 | 실시간 조율·산출물 상호 참조 필요 |

## 에이전트 구성 (전문가 풀)

| 전문가 | 담당 영역 | 스킬 | 트리거 예시 |
|--------|----------|------|-------------|
| `site-manager` | GitHub Pages 프로필 사이트 | `site-content-update` | 사진 추가, 소식 문구, 프로필/링크 수정 |
| `bot-developer` | 카카오 오픈빌더 스킬 서버 | `kakao-skill-server` | 검증 로직, 상담원 알림, 시나리오 문구 |
| `deploy-ops` | 배포·환경변수·백업 동기화 | `render-deploy-ops` | Render 배포, 장애, 백업 상태 점검, 하네스 설정 백업 |
| `qa-reviewer` | 코드/콘텐츠 수정 직후 정합성 검증 | `changyeon-qa` | server.js↔SCENARIO.md 대조, 회귀 테스트, index.html 무결성 |

## 워크플로우

### Phase 0: 컨텍스트 확인 (후속 작업 지원)

1. `.claude/agents/`, `.claude/skills/`, `CLAUDE.md`, `_workspace/`(있다면) 확인
2. 사용자 요청이 다음 중 무엇인지 판단:
   - **단일 영역 요청** → Phase 1(단일 전문가 배정)로 진행
   - **교차 영역 요청** → Phase 2(팀 구성)로 진행
   - **하네스 자체의 점검/수정 요청** ("에이전트 추가해줘", "하네스 점검해줘") → harness 스킬의 Phase 7-5 운영/유지보수 워크플로우로 위임
3. `_workspace/`에 이전 점검·작업 기록이 있으면 먼저 Read하여 맥락을 파악한다

### Phase 1: 단일 전문가 배정 (기본 경로 — 대부분의 요청)

1. 요청 키워드로 담당 전문가 판단 (위 표 참조). 애매하면 사용자에게 한 줄로 확인
2. `Agent(subagent_type: "{agent-name}", model: "opus", prompt: "{요청 내용 + 관련 파일 경로}")` 호출
   - 사이트 관련: `changyeon4277.github.io/` 경로를 프롬프트에 명시
   - 챗봇 관련: `changyeon-kakao-bot/` 경로를 프롬프트에 명시
   - 배포/백업 관련: 3개 저장소 경로 모두 명시
3. **`site-manager` 또는 `bot-developer`가 코드/콘텐츠를 실제로 변경했으면**, 결과 보고 전에 반드시 `Agent(subagent_type: "qa-reviewer", model: "opus", prompt: "{무엇이 바뀌었는지 + 관련 파일 경로}")`를 호출해 점진적 검증을 거친다. 단순 조회·상태 확인처럼 실제 변경이 없었던 요청은 QA를 건너뛴다
4. QA 판정이 FIX/REDO면 담당 전문가에게 재작업을 1회 요청하고 재검증, PASS면 그대로 다음 단계 진행
5. 결과를 사용자에게 요약 보고 (QA 판정 포함)

### Phase 2: 교차 영역 요청 (팀 구성)

예: "새 상담 메뉴 만들고 사이트에도 반영하고 배포까지 해줘" 같이 여러 전문가의 협업이 필요한 경우.

1. `TeamCreate(team_name: "changyeon-team", members: [site-manager, bot-developer, deploy-ops, qa-reviewer 중 필요한 조합])` — 코드/콘텐츠 수정이 포함된 교차 영역 요청은 `qa-reviewer`를 항상 포함시킨다
2. `TaskCreate`로 작업 분배 — 의존성을 명시한다: 배포 작업은 코드 완료 후(`depends_on`), QA 작업은 해당 담당자의 산출물 완료 직후(모듈별 점진적 QA, 전체 완료 후 1회가 아님)
3. 팀원들이 `SendMessage`로 조율 (예: bot-developer가 새 환경변수를 추가하면 deploy-ops에게 알림, qa-reviewer가 FIX 판정 시 담당자에게 직접 알림)
4. 모든 작업과 QA 판정(PASS)까지 완료된 후 결과 종합, `TeamDelete`로 팀 정리

### Phase 3: 결과 보고 및 백업 리마인드

모든 경로 공통 — 작업이 끝나면:
1. 변경된 저장소에 커밋되지 않은 내용이 남아있는지 확인 (`deploy-ops`의 점검 로직 활용)
2. 남아있으면 사용자에게 push까지 마무리할지 확인 (노트북 백업이 안 되는 환경이므로, 커밋·푸시가 곧 백업)

## 데이터 흐름

```
[사용자 요청]
    ↓
[changyeon-ops: 영역 판단]
    ├─ 단일 영역 → Agent(전문가 1명, model: opus) → (코드/콘텐츠 변경 시) Agent(qa-reviewer) → 결과 반환
    └─ 교차 영역 → TeamCreate → 전문가 2~4명(qa-reviewer 포함) SendMessage 조율 → 종합 → TeamDelete
    ↓
[Phase 3: 커밋/푸시 상태 확인 및 리마인드]
```

## 에러 핸들링

| 상황 | 전략 |
|------|------|
| 전문가 에이전트 실행 실패 | 1회 재시도, 재실패 시 사용자에게 원인(에러 메시지) 그대로 전달 |
| 영역 판단이 애매함 | 임의로 추측하지 않고 사용자에게 "사이트 쪽인가요, 챗봇 쪽인가요?" 한 줄로 확인 |
| 팀원 간 작업 충돌 (예: 같은 파일 동시 수정) | 나중 작업자가 먼저 `git status`로 충돌 여부 확인 후 순차 처리로 전환 |
| `qa-reviewer`가 FIX/REDO 판정 | 담당 전문가에게 재작업 1회 요청 → 재검증. 재실패 시 사용자에게 구체적 불일치 내용 그대로 보고 (QA가 직접 고치지 않음) |
| `changyeon` 저장소(하네스 백업소) 자체를 수정해야 하는 요청 | `deploy-ops`의 "하네스 설정 백업" 절차로 위임 |

## 테스트 시나리오

### 정상 흐름 (단일 영역)
1. 사용자: "사진 3장 올렸어요, 사이트에 반영해주세요"
2. Phase 0: 단일 영역(사이트)으로 판단
3. Phase 1: `site-manager` 서브 에이전트 호출 → `node update-site.js` 실행 → 커밋/푸시
4. Phase 1-3: `qa-reviewer` 호출 → 이미지 참조·news-scroll 마커 무결성 확인 → PASS
5. Phase 3: push 완료 확인, 사용자에게 반영 시점 안내 (QA PASS 포함)

### 에러 흐름 (교차 영역 + 부분 실패)
1. 사용자: "예약 검증에 이메일 항목 추가하고, 사이트 문의 폼 문구도 바꾸고, 배포까지 해주세요"
2. Phase 0: 교차 영역으로 판단 → Phase 2 진입
3. `TeamCreate(changyeon-team, [bot-developer, site-manager, deploy-ops])`
4. bot-developer가 서버 로직 수정 중 실패(예: 검증 정규식 오류) → 팀 내 SendMessage로 상태 공유
5. site-manager 작업은 정상 완료, deploy-ops는 bot-developer 완료 대기 중 타임아웃
6. 리더가 실패한 bot-developer 작업만 재시도 → 재성공 시 deploy-ops에게 배포 재개 알림
7. 최종 보고: 어느 부분이 지연/재시도됐는지 명시하고 종합 결과 전달

## 후속 작업 지원

"저번에 올린 사진 문구 다시 바꿔줘", "챗봇 검증 로직 방금 고친 거 되돌려줘", "배포 다시 확인해줘" 등은 모두 이 스킬이 트리거되어야 한다. Phase 0에서 `git log`/`_workspace/` 확인으로 이전 맥락을 파악한 뒤 해당 전문가에게 배정한다.
