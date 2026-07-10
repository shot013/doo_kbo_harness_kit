# doo_kbo_harness_kit

`doo_kbo_app`에서 실제로 사용하며 검증된 Claude Code 하네스 엔지니어링 패턴(규칙,
스킬, 훅, 서브에이전트, CI 연동)을 다른 프로젝트에도 재사용할 수 있도록 일반화한
템플릿 모음입니다. 특정 언어/프레임워크(Flutter 등)에 종속된 내용은 모두
`{{PLACEHOLDER}}` 형태로 빼두었습니다.

## 왜 이런 구조인가

Claude Code는 다음 네 가지 층으로 저장소별 작업 방식을 학습합니다. 이 키트는
그 네 층을 프로젝트에 새로 세팅할 때 매번 처음부터 설계하지 않도록 뼈대를 제공합니다.

| 층 | 위치 | 적용 시점 |
|---|---|---|
| 규칙 | `CLAUDE.md`, `.claude/rules/*.md` | 항상 (세션 시작 시 자동 로드) |
| Skills | `.claude/skills/*/SKILL.md` | 관련 작업 시 자동, 또는 `/name`으로 직접 호출 |
| Hooks | `.claude/settings.json`, `.claude/hooks/*.sh` | 완전 자동 (파일 저장 등 이벤트 훅) |
| Subagent | `.claude/agents/*.md` | 필요 시 자동 위임, 또는 직접 요청 |

## 적용 방법

1. 이 저장소의 파일을 새 프로젝트 루트에 복사한다 (`.template`/`.example` 접미사 포함 파일 전체).
2. 아래 플레이스홀더 표를 참고해 프로젝트에 맞는 값으로 전역 치환한다.
3. `.template` 접미사를 제거하고, `.claude/settings.local.json.example`은
   `.claude/settings.local.json`으로 복사한 뒤 개인 환경에 맞게 수정한다 (이 파일은
   `.gitignore`에 이미 등록되어 있음 — 팀 공유 설정은 `settings.json`에만 둔다).
4. `.claude/rules/architecture.md.template`, `.claude/skills/scaffold-module/SKILL.md.template`은
   실제 아키텍처 패턴(레이어 이름, 에러 처리 방식 등)에 맞게 내용을 다시 쓴다 — 이 두 파일은
   치환만으로 끝나지 않고 프로젝트 고유의 설계를 반영해야 하는 파일이다.
5. `.claude/agents/code-reviewer.md.template`을 프로젝트 컨벤션에 맞춰 구체화한다.
6. `flutter-reviewer`처럼 이름 자체를 프로젝트/스택에 맞게 바꾼다 (예: `ts-reviewer`).

## 플레이스홀더 목록

| 플레이스홀더 | 의미 | doo_kbo_app 예시 |
|---|---|---|
| `{{PROJECT_NAME}}` | 프로젝트/저장소 이름 | `doo_kbo_app` |
| `{{LANGUAGE_STACK}}` | 언어/프레임워크 한 줄 요약 | `Dart/Flutter (Riverpod, go_router, dio, equatable)` |
| `{{INSTALL_CMD}}` | 의존성 설치 명령 | `flutter pub get` |
| `{{RUN_CMD}}` | 앱 실행 명령 | `flutter run` |
| `{{FORMAT_CHECK_CMD}}` | 포맷 체크(비파괴) 명령 | `dart format --output=none --set-exit-if-changed .` |
| `{{FORMAT_FIX_CMD}}` | 포맷 자동 적용 명령 | `dart format .` |
| `{{FORMAT_FIX_FILE_CMD}}` | 파일 하나만 포맷하는 명령 (훅용, `$f` 인자) | `dart format "$f"` |
| `{{FORMATTER_BIN}}` | 포맷터 실행 파일명 (PATH 체크용) | `dart` |
| `{{LINT_CMD}}` | 정적 분석 명령 | `flutter analyze` |
| `{{TEST_CMD}}` | 전체 테스트 명령 | `flutter test` |
| `{{TEST_SINGLE_CMD}}` | 단일 파일 테스트 명령 | `flutter test <path>` |
| `{{TEST_NAME_CMD}}` | 이름으로 단일 테스트 실행 | `flutter test <path> --plain-name "<name>"` |
| `{{SRC_DIR}}` | 소스 루트 | `lib/` |
| `{{TEST_DIR}}` | 테스트 루트 | `test/` |
| `{{SRC_EXT}}` | 소스 파일 확장자 | `dart` |
| `{{GENERATED_DIR_PATTERN}}` | 수정 차단할 생성 파일 디렉터리 패턴 (`case` 문법) | `*/build/*\|*/.dart_tool/*` |
| `{{CI_SETUP_ACTION}}` | CI에서 툴체인 설치에 쓰는 GitHub Action | `subosito/flutter-action@v2` |
| `{{CI_SETUP_WITH}}` | 위 액션의 `with:` 옵션 | `channel: stable`<br>`cache: true` |
| `{{ARCHITECTURE_SUMMARY}}` | 아키텍처 개요 (CLAUDE.md용) | Clean Architecture feature-first 설명 |
| `{{TEAM_SIZE_CONTEXT}}` | 협업 규칙의 근거가 되는 팀 규모/맥락 | `6인 이상 팀` |
| `{{LINT_RULE_SUMMARY}}` | 활성화한 주요 린트 규칙 요약 | strict-casts 등 |
| `{{LINT_CONFIG_FILE}}` | 린트 설정 파일명 | `analysis_options.yaml` |
| `{{TEMPLATE_MODULE_PATH}}` | 새 모듈 작성 시 복사할 템플릿 모듈 경로 | `features/example/` |
| `{{REVIEWER_AGENT_NAME}}` | 리뷰 서브에이전트 이름 (스택에 맞게) | `flutter-reviewer` |

## 파일별 역할

- `CLAUDE.md.template` — 매 세션 자동 로드되는 최상위 가이드 (명령어, 아키텍처 요약).
- `CONTRIBUTING.md.template` — 브랜치/커밋/PR 규칙 (사람 + Claude 공용).
- `MEMORY.md`, `ERRORS.md` — 코드/git 히스토리로 알 수 없는 맥락과 재발 에러를 쌓는 빈 템플릿. 내용 자체는 프로젝트 중립적이라 치환 없이 그대로 복사.
- `.claude/settings.json.template` — 팀 공유 훅/권한 설정.
- `.claude/settings.local.json.example` — 개인용 권한 오버라이드 예시 (`.gitignore` 대상).
- `.claude/hooks/*.sh.template` — 저장 시 자동 포맷 / 생성 파일 수정 차단 훅.
- `.claude/rules/*.md` — 아키텍처/코드스타일/git 규칙. `git-workflow.md.template`는 언어 중립적이지만 린트/테스트 명령 플레이스홀더 2개만 치환하면 그대로 사용 가능.
- `.claude/skills/verify/SKILL.md.template` — CI와 동일한 순서로 로컬 검증.
- `.claude/skills/scaffold-module/SKILL.md.template` — 템플릿 모듈을 복사해 새 기능을 만드는 절차 (아키텍처에 맞게 재작성 필요).
- `.claude/agents/code-reviewer.md.template` — 프로젝트 고유 컨벤션 리뷰 서브에이전트.
- `.github/workflows/ci.yaml.template` — format → lint → test 3단계 CI.
- `.github/PULL_REQUEST_TEMPLATE.md` — 언어 중립적이라 그대로 사용 가능.
- `docs/CLAUDE_CODE.md.template` — 위 네 층이 어떻게 맞물려 자동으로 돌아가는지 팀원에게 설명하는 문서.
