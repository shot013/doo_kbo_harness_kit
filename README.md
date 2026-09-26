# doo_kbo_harness_kit

`doo_kbo_scraping_server`에서 실제로 사용하며 검증된 Claude Code 하네스 엔지니어링 패턴(규칙,
스킬, 훅, 서브에이전트, CI 연동)을 다른 프로젝트에도 재사용할 수 있도록 일반화한
템플릿 모음입니다. 특정 언어/프레임워크(Node.js/Nest.js 등)에 종속된 내용은 모두
`{{PLACEHOLDER}}` 형태로 빼두었습니다.

## 왜 이런 구조인가

Claude Code는 다음 네 가지 층으로 저장소별 작업 방식을 학습합니다. 이 키트는
그 네 층을 프로젝트에 새로 세팅할 때 매번 처음부터 설계하지 않도록 뼈대를 제공합니다.

| 층 | 위치 | 적용 시점 | 상시 토큰 비용 |
|---|---|---|---|
| 규칙 | `CLAUDE.md` + 거기서 `@`로 import하는 `.claude/rules/*.md` | 항상 (매 세션·매 턴) | 전액 선불 |
| Skills | `.claude/skills/*/SKILL.md` | 관련 작업 시 자동, 또는 `/name`으로 직접 호출 | `description`만 |
| Hooks | `.claude/settings.json`, `.claude/hooks/*.sh` | 완전 자동 (도구 호출 이벤트) | 0 |
| Subagent | `.claude/agents/*.md` | 필요 시 자동 위임, 또는 직접 요청 | `description`만 |

네 층의 차이는 "언제 적용되나"만이 아니라 **비용**입니다. 규칙은 매 턴 과금되고, 훅은
공짜이며, 스킬과 서브에이전트는 호출될 때만 본문 값을 냅니다. 같은 규칙이라도 어느 층에
두느냐에 따라 세션 비용이 크게 달라지므로, 배치 기준은 `docs/CLAUDE_CODE.md`의
"어디에 둘 것인가"를 따르세요.

## 파일별 역할

- `CLAUDE.md.template` — 매 세션 자동 로드되는 최상위 가이드 (명령어, 아키텍처 요약).
- `CONTRIBUTING.md.template` — 브랜치/커밋/PR 규칙 (사람 + Claude 공용).
- `MEMORY.md`, `ERRORS.md` — 코드/git 히스토리로 알 수 없는 맥락과 재발 에러를 쌓는 빈 템플릿. 내용 자체는 프로젝트 중립적이라 치환 없이 그대로 복사. **계속 자라는 문서이므로 `CLAUDE.md`에서 `@`로 import하지 않는다** — 필요할 때 grep해서 해당 부분만 읽는다.
- `.claude/settings.json.template` — 팀 공유 훅/권한 설정.
- `.claude/settings.local.json.example` — 개인용 권한 오버라이드 예시 (`.gitignore` 대상).
- `.claude/hooks/format-on-save.sh.template` — 저장 시 자동 포맷.
- `.claude/hooks/block-generated-edit.sh.template` — 생성 파일 수정 차단.
- `.claude/hooks/guard-git.sh.template` — `main`/`master` 직접 push, 브랜치명 규약 위반, Conventional Commits가 아닌 커밋 메시지를 차단. git 규약 중 기계적으로 판정 가능한 부분을 프롬프트가 아니라 훅으로 강제해 상시 토큰 비용을 0으로 만든다. 파싱 실패 시 통과(fail-open).
- `.claude/rules/*.md` — 매 턴 과금되는 자리. 아키텍처/코드스타일/판단이 필요한 git 규칙만 남긴다. `CLAUDE.md`가 `@`로 명시적으로 import하므로, 규칙 파일을 추가하면 `CLAUDE.md`에도 한 줄 추가해야 한다.
- `.claude/skills/verify/SKILL.md.template` — CI와 동일한 순서로 로컬 검증.
- `.claude/skills/scaffold-module/SKILL.md.template` — 템플릿 모듈을 복사해 새 기능을 만드는 절차 (아키텍처에 맞게 재작성 필요).
- `.claude/skills/api-docs-sync/SKILL.md.template` — 백엔드 API 변경 시 관련 문서를 같은 PR에서 함께 업데이트하도록 강제. 조건부 규칙이라 rules가 아니라 skill에 둔다 — API를 건드리지 않는 세션에서는 `description` 외에 아무 비용도 들지 않는다.
- `.claude/agents/code-reviewer.md.template` — 프로젝트 고유 컨벤션 리뷰 서브에이전트.
- `.github/workflows/ci.yaml.template` — format → lint → test 3단계 CI.
- `.github/PULL_REQUEST_TEMPLATE.md` — 언어 중립적이라 그대로 사용 가능.
- `docs/CLAUDE_CODE.md.template` — 위 네 층이 어떻게 맞물려 자동으로 돌아가는지 팀원에게 설명하는 문서.
- `docs/RULE_APPLICATION_ORDER.md.template` — 위 네 층이 세션 시작 → 코딩 → 커밋 전 → PR → CI 중 어느 시점에, 어떤 순서로 발동되는지 정리한 문서. 프론트/서버/DB/인프라가 레이어로 분리된 프로젝트라면 확장하는 방법도 포함.
- `docs/TEMPLATE_EXAMPLES.md.template` — 각 템플릿 파일을 어떻게 채우는지 보여주는 예시 모음. 예시를 규칙 파일 안에 주석으로 두면 사람에게만 안 보일 뿐 모델에게는 그대로 전달되어 매 턴 과금되므로, 컨텍스트에 로드되지 않는 이 문서로 모아두었다.

## 적용 방법

1. 이 저장소의 파일을 새 프로젝트 루트에 복사한다 (`.template`/`.example` 접미사 포함 파일 전체).
2. 아래 플레이스홀더 표를 참고해 프로젝트에 맞는 값으로 전역 치환한다.
3. `.template` 접미사를 제거하고, `.claude/settings.local.json.example`은
   `.claude/settings.local.json`으로 복사한 뒤 개인 환경에 맞게 수정한다 (이 파일은
   `.gitignore`에 이미 등록되어 있음 — 팀 공유 설정은 `settings.json`에만 둔다).
4. `.claude/rules/architecture.md.template`, `.claude/skills/scaffold-module/SKILL.md.template`은
   실제 아키텍처 패턴(레이어 이름, 에러 처리 방식 등)에 맞게 내용을 다시 쓴다 — 이 두 파일은
   치환만으로 끝나지 않고 프로젝트 고유의 설계를 반영해야 하는 파일이다.
   작성 예시는 `docs/TEMPLATE_EXAMPLES.md`에 있다.
5. `.claude/agents/code-reviewer.md.template`을 프로젝트 컨벤션에 맞춰 구체화한다.
6. `nest-reviewer`처럼 이름 자체를 프로젝트/스택에 맞게 바꾼다 (예: `ts-reviewer`).
7. 세션을 열고 `/context`로 `CLAUDE.md`와 규칙이 실제로 로드됐는지, 상시 비용이 얼마인지
   확인한다. 예상보다 크다면 `docs/CLAUDE_CODE.md`의 "어디에 둘 것인가" 순서대로
   훅 → 스킬 → 서브에이전트로 내려보낼 수 있는 항목이 남아 있는지 점검한다.

## 플레이스홀더 목록

| 플레이스홀더 | 의미 | doo_kbo_app 예시 |
|---|---|---|
| `{{PROJECT_NAME}}` | 프로젝트/저장소 이름 | `doo_kbo_scraping_server` |
| `{{LANGUAGE_STACK}}` | 언어/프레임워크 한 줄 요약 | `Node.js/Nest.js (TypeScript)` |
| `{{INSTALL_CMD}}` | 의존성 설치 명령 | `npm install` |
| `{{RUN_CMD}}` | 앱 실행 명령 | `npm run start:dev` |
| `{{FORMAT_CHECK_CMD}}` | 포맷 체크(비파괴) 명령 | `npx prettier --check .` |
| `{{FORMAT_FIX_CMD}}` | 포맷 자동 적용 명령 | `npx prettier --write .` |
| `{{FORMAT_FIX_FILE_CMD}}` | 파일 하나만 포맷하는 명령 (훅용, `$f` 인자) | `npx prettier --write "$f"` |
| `{{FORMATTER_BIN}}` | 포맷터 실행 파일명 (PATH 체크용) | `prettier` |
| `{{LINT_CMD}}` | 정적 분석 명령 | `npm run lint` |
| `{{TEST_CMD}}` | 전체 테스트 명령 | `npm run test` |
| `{{TEST_SINGLE_CMD}}` | 단일 파일 테스트 명령 | `npm run test -- <path>` |
| `{{TEST_NAME_CMD}}` | 이름으로 단일 테스트 실행 | `npm run test -- <path> --testNamePattern "<name>"` |
| `{{SRC_DIR}}` | 소스 루트 | `src/` |
| `{{TEST_DIR}}` | 테스트 루트 | `test/` |
| `{{SRC_EXT}}` | 소스 파일 확장자 | `ts` |
| `{{GENERATED_DIR_PATTERN}}` | 수정 차단할 생성 파일 디렉터리 패턴 (`case` 문법) | `*/dist/*\|*/coverage/*` |
| `{{CI_SETUP_ACTION}}` | CI에서 툴체인 설치에 쓰는 GitHub Action | `actions/setup-node@v4` |
| `{{CI_SETUP_WITH}}` | 위 액션의 `with:` 옵션 | `node-version: 20`<br>`cache: npm` |
| `{{ARCHITECTURE_SUMMARY}}` | 아키텍처 개요 (CLAUDE.md용) | Modular Nest.js architecture with controllers, services, modules, and repositories |
| `{{TEAM_SIZE_CONTEXT}}` | 협업 규칙의 근거가 되는 팀 규모/맥락 | `3~8인 팀` |
| `{{LINT_RULE_SUMMARY}}` | 활성화한 주요 린트 규칙 요약 | `@typescript-eslint/*`, import/order, no-explicit-any |
| `{{LINT_CONFIG_FILE}}` | 린트 설정 파일명 | `eslint.config.js` |
| `{{TEMPLATE_MODULE_PATH}}` | 새 모듈 작성 시 복사할 템플릿 모듈 경로 | `modules/example/` |
| `{{REVIEWER_AGENT_NAME}}` | 리뷰 서브에이전트 이름 (스택에 맞게) | `nest-reviewer` |
| `{{API_DOC_PATH}}` | 백엔드 API 변경 시 함께 갱신해야 하는 API 문서 경로 | `docs/API.md` (또는 Swagger UI 경로) |
