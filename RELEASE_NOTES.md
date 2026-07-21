# 2026-07-22-snapshot

이 릴리스는 정식 제품 릴리스가 아닌 Linux amd64 사전 테스트 스냅샷입니다.
대응 소스 commit은
`017fd7331749ee0dfa0587119de71e38343b1e7a`입니다.

## Provider replay와 장시간 실행

- Codex Responses의 provider-owned replay state를 로컬, 서버, streaming 및
  `/resume` 경로에서 보존합니다.
- replay state가 없는 오래된 tool history와 다른 provider/model의 state는 원문
  인자·결과 없이 제한된 durable summary로 변환하여 변경 도구의 중복 실행을
  방지합니다.
- 인증, 설정, 프로토콜 오류는 즉시 반환하고 전송 전 실패나 명확한 transient
  오류만 제한적으로 재시도합니다.
- interactive turn은 기본적으로 전체 시간 제한이 없습니다. `turn_timeout` 또는
  `--timeout`을 지정할 수 있으며 first-event/read-idle/tool timeout은 독립적으로
  적용됩니다.

## Durable orchestration과 Skill optimization

- SQLite `BUSY_SNAPSHOT` 발생 시 dispatch 상태 전이의 전체 read/CAS/write
  transaction을 bounded backoff로 다시 수행합니다.
- Lua workflow와 Galley schema 30을 사용하는 `skill.optimize`를 추가했습니다.
  실행, phase, 후보, 평가 및 evidence를 재시작 후에도 복구할 수 있습니다.
- 실제 optimizer/evaluator와 검증 suite가 배선되지 않은 환경에서는 promotion을
  성공으로 가장하지 않고 fail closed로 종료합니다.
- child agent가 별도 모델을 지정하지 않으면 현재 활성 모델을 상속합니다.

## Provider와 설정

- Qwen Token Plan provider, reasoning effort 설정 및 streaming usage 수집을
  지원합니다.
- 외부 OAuth credential 파일 변경을 감지해 Codex와 Grok credential을 안전하게
  다시 로드합니다.
- `/config auto_planmode`와 `turn_timeout` 설정을 추가했습니다.

## 기동 및 작업공간

- 심볼릭 링크 작업공간을 시작 시 canonical directory로 고정합니다.
- private tool output buffer의 `O_NOFOLLOW` 보안 경계는 유지됩니다.
- `~/ontology -> /docker/ontology`와 같은 작업공간에서 일반 `jikjicode` 명령으로
  대화형 TUI를 시작할 수 있습니다.
- 비 Git 디렉터리의 대화형 세션은 허용하지만 positional goal과 `exec` one-shot은
  `--skip-git-repo-check`를 명시하지 않는 한 기존 Git 안전 검사를 유지합니다.

## MCP 및 Ontology

- managed MCP server별 lifecycle 작업을 직렬화하고 close barrier를 적용했습니다.
- `/config`의 MCP enable/disable은 TUI를 막지 않고 진행 상태와 결과를 표시합니다.
- ontology graph와 provenance를 원자적으로 게시하고 embedding cache를 포함한 실제
  메모리 사용량을 status snapshot에 반영합니다.
- 실제 Jikji stdio MCP server를 연결하여 도구 발견과 실행을 검증했습니다.

## Full 바이너리 데이터 연결

- DuckDB를 통해 PostgreSQL과 MySQL 외부 데이터를 구조화된
  `duckdb.read`/`duckdb.insert` 도구로 연결할 수 있습니다.
- connector는 tenant allowlist, `read_only`/`read_write` 모드 및 환경 변수 기반
  DSN만 허용합니다. 임의 `ATTACH` 또는 임의 SQL은 노출하지 않습니다.
- `/duckdb`에서 secret-free connector 상태를 확인할 수 있습니다.
- Jikjicode의 선택적 `sqlguard` heuristic overlay는 기본 `off`입니다. 이를 꺼도
  SELECT-only 검증, tenant scoping, stacked statement 및 외부 작업 차단,
  DuckDB sandbox는 유지됩니다.

## Self-update

- `jikjicode update --check`, `jikjicode update`, `--version`, `--force`를
  지원합니다.
- 대화형 터미널은 최대 24시간에 한 번, 최대 5초의 비동기 확인을 수행합니다.
  오프라인 오류는 기동을 막지 않으며 자동으로 바이너리를 설치하지 않습니다.
- 현재 standard/full flavor와 OS/architecture가 일치하는 자산만 선택합니다.
- 바이너리에 pre-test RSA 공개키와 SPKI SHA-256 지문을 고정했습니다.
  `SHA256SUMS.sig`, 릴리스 공개키 사본, GitHub asset digest와 파일 SHA-256이
  모두 일치한 파일만 동기화 후 원자 교체합니다.
- 서명된 `release-metadata.json`이 저장소, 태그, 버전과 대응 소스 commit을
  결속하여 과거의 정상 서명 번들을 더 높은 가짜 버전으로 재생할 수 없게 합니다.
- GitHub API와 다운로드 URL의 HTTPS/host/redirect 경계를 검증하고 인증 토큰을
  교차 호스트로 전달하지 않습니다.
- prerelease는 게시 시각이 아니라 버전을 우선해 선택하며, `dev`, `local`,
  `dirty` 빌드는 `--force` 없이는 설치 대상으로 사용하지 않습니다.
- `JIKJICODE_NO_UPDATE_CHECK=1`은 백그라운드 확인만 비활성화합니다.

## 검증

- Jikji 전체 `make verify`
- Jikjicode 관련 race detector
- PostgreSQL 17 및 MySQL 8.4 임시 컨테이너를 사용한 실제 DuckDB connector 통합
- 심볼릭 링크 및 비 Git 작업공간에서 실제 PTY TUI 기동
- 실제 GitHub prerelease 조회를 사용한 `update --check`
- Linux, macOS, Windows 테스트 교차 컴파일과 FreeBSD/OpenBSD portability build

## 포함 파일

- `jikjicode-linux-amd64`: 일반 정적 빌드
- `jikjicode-full-linux-amd64`: DuckDB 포함 확장 빌드
- `SHA256SUMS`: 배포 파일 무결성 확인
- `SHA256SUMS.sig`, `release-signing-public.pem`: 배포 체크섬 인증
- `release-metadata.json`: 릴리스 identity와 대응 소스 결속
- `LICENSE`, `LICENSING.md`, `SOURCE_OFFER.md`: 라이선스 및 대응 소스 안내

Full 빌드는 내장 CPython을 포함하지 않습니다.

## 테스트된 Provider

- Grok OAuth
- GPT / Codex OAuth
- Ollama Pro / Cloud
- GLM (Z.AI) Coding Plan

## 주의

- 프로덕션 배포용 정식 릴리스가 아닙니다.
- 설정과 로그에 API key, OAuth token 또는 실제 DSN을 기록하지 마십시오.
- 기본 라이선스는 GPL-3.0-only입니다. 대응 소스 요청은 `SOURCE_OFFER.md`를
  따르십시오.
