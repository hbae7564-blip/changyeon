---
name: changyeon-qa
description: "site-manager 또는 bot-developer가 changyeon4277.github.io나 changyeon-kakao-bot의 코드/콘텐츠를 수정한 직후 정합성을 검증. server.js 응답 문구와 SCENARIO.md 대조, README의 5개 회귀 테스트 케이스 재실행, index.html 이미지 참조와 news-scroll 마커 무결성 확인, package.json/.env.example 경계면 점검을 수행. '검증해줘', '테스트해줘', 'QA', '제대로 됐는지 확인' 요청 시, 그리고 changyeon-ops 오케스트레이터가 코드/콘텐츠 수정 작업 직후 자동으로 반드시 사용."
---

# Changyeon QA — 점진적 경계면 검증

이 스킬은 `qa-reviewer` 에이전트가 사용한다. 목적은 "파일이 생성됐는가"가 아니라 "서로 맞아야 할 두 지점이 실제로 맞는가"를 확인하는 것이다.

## 카카오봇 회귀 테스트 (bot-developer 수정 직후)

1. `changyeon-kakao-bot` 폴더에서 `npm install && npm start` (이미 실행 중이면 생략)
2. `http://localhost:3000`에서 "정상적으로 실행 중입니다" 확인
3. `/skill/validate`에 아래 5개 케이스를 오픈빌더 요청 형식으로 POST하여 응답 확인 (README.md 기준 케이스, `curl` 또는 간단한 스크립트 사용):

| # | 입력 | 기대 결과 |
|---|------|----------|
| 1 | 정상 4개 항목 | 요약 메시지 + "다시입력하기" 버튼 |
| 2 | 전화번호 `01012345678a` | 오류 메시지 + "처음부터 다시입력하기" 버튼 |
| 3 | 성함 `ㅁㄴㅇㄹ` 또는 문의내용 `!!!` | 오류 메시지 |
| 4 | 문의내용에 `"번호는 010-2222-3333이에요"` 포함 | 정상 통과 |
| 5 | 발화 `"처음부터 다시입력하기"` | 파라미터 초기화 응답 (context lifeSpan: 0) |

4. **응답문구 대조**: 위에서 받은 실제 응답 텍스트가 `SCENARIO.md`에 적힌 문구와 일치하는지 두 파일을 나란히 읽고 비교한다. 사소한 표현 차이도 FIX 대상이다 — 사용자가 오픈빌더 화면에 SCENARIO.md 문구를 그대로 복사해 넣기 때문에, 실제 코드 응답과 문서가 다르면 운영 중 혼선이 생긴다.
5. **5초 제한 정적 검사**: `notifyCounselor(...)` 호출부에 `await`가 붙어있지 않고 `.catch()`로 끝나는지 코드를 직접 읽어 확인한다. `await`가 붙어있으면 무조건 REDO.

## 사이트 정합성 검사 (site-manager 수정 직후)

1. `index.html`에서 `src="..."` 패턴을 모두 추출
2. 각 경로가 `changyeon4277.github.io/` 폴더에 실제 파일로 존재하는지 확인 (깨진 이미지 링크 검출)
3. `<div class="news-scroll">`와 그 닫는 `</div>\n</section>` 구조가 원형대로 남아있는지 확인 — 이 마커가 손상되면 다음 `update-site.js` 실행이 "영역을 찾지 못했습니다" 에러로 실패한다
4. 새로 추가된 사진이 있다면, 실제로 `news-card` 블록으로 등록됐는지 (파일 존재 + HTML 등록 둘 다) 확인

## 배포 설정 경계면 검사 (deploy-ops 작업 또는 정기 점검 시)

1. `package.json`의 `scripts.start`가 가리키는 파일 경로가 실제로 존재하고, 그 파일에 `app.listen(`이 있는지 확인
2. `server.js` 전체에서 `process.env.` 로 참조되는 모든 변수명을 추출하고, `.env.example`에 전부 등록되어 있는지 대조 — 누락되면 사용자가 Render에 뭘 입력해야 할지 알 수 없게 된다

## 판정 후 처리

- **PASS**: 오케스트레이터에게 통과 보고, 다음 단계(배포/커밋) 진행 가능
- **FIX**: 구체적 수정 지시와 함께 담당 에이전트(`bot-developer`/`site-manager`)에게 되돌림 — QA가 직접 고치지 않는다
- **REDO**: 로직 재설계가 필요함을 명시하고, 5초 제한처럼 절대 원칙을 위반한 경우 그 원칙을 다시 인용하여 설명

## 회귀 테스트 스크립트 재사용

같은 5개 케이스를 매번 손으로 만들지 않도록, 첫 실행 시 `changyeon-kakao-bot/_qa/regression-cases.json`에 위 표를 JSON으로 저장해두고 이후 검증부터는 이 파일을 읽어 재사용한다 (파일이 없으면 새로 만든다).
