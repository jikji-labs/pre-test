# Jikjicode 사전 테스트 스냅샷

이 저장소는 Jikjicode의 사전 공개 테스트 바이너리와 실행 안내를 제공합니다.
현재 배포본은 **정식 릴리스가 아니며 프로덕션 사용을 보장하지 않습니다.**
소스 코드는 이 저장소에 포함하지 않습니다.

## 2026-07-20 스냅샷

지원 환경은 Linux x86-64(amd64)입니다.

| 파일 | 구성 |
|---|---|
| `jikjicode-linux-amd64` | 일반 정적 바이너리 |
| `jikjicode-full-linux-amd64` | DuckDB 분석 엔진을 포함한 확장 정적 바이너리 |
| `SHA256SUMS` | 배포 파일 SHA-256 체크섬 |

Full 바이너리는 이번 스냅샷에서 DuckDB를 포함합니다. 내장 CPython은 아직 포함하지
않습니다. Python 도구는 시스템에 `python3`가 설치된 경우 두 바이너리 모두 외부
프로세스 방식으로 사용할 수 있습니다.

## 설치

릴리스 페이지에서 원하는 바이너리와 `SHA256SUMS`를 받은 뒤 무결성을 확인합니다.

```bash
sha256sum -c SHA256SUMS --ignore-missing
install -m 0755 jikjicode-linux-amd64 ~/.local/bin/jikjicode
jikjicode version
```

Full 바이너리를 설치하려면 두 번째 줄의 파일명을
`jikjicode-full-linux-amd64`로 바꿉니다.

## 최초 설정

대화형 설정 마법사가 가장 안전한 시작 방법입니다.

```bash
jikjicode init
jikjicode doctor
jikjicode
```

설정 파일은 기본적으로 `~/.jikji/jikjicode.yaml`을 사용합니다. 이 저장소의
[`jikjicode.yaml`](./jikjicode.yaml)은 비밀값을 포함하지 않는 예제입니다.
API 키 값은 YAML에 직접 쓰지 말고 환경 변수 이름만 `api_key_env`에 지정하십시오.
`--config <file>`을 명시하면 전역 및 디렉터리별 자동 탐색 대신 해당 파일만
사용합니다.

현재 작업 디렉터리에 `.jikji/jikjicode.yaml`이 있으면 전역 설정보다 높은 우선순위로
안전한 실행 옵션을 좁혀서 덮어쓸 수 있습니다. 우선순위는 다음과 같습니다.

```text
CLI 옵션 > <현재 디렉터리>/.jikji/jikjicode.yaml > ~/.jikji/jikjicode.yaml > 기본값
```

프로젝트 설정은 상위 디렉터리를 탐색하지 않으며 provider, credential, env 파일을
바꿀 수 없습니다.

## 주요 실행 명령

```bash
jikjicode                         # TTY 전체 화면 UI
jikjicode "이 저장소를 검토해줘"   # 한 번 실행
jikjicode exec "테스트를 실행해줘"
jikjicode tools                   # 활성 도구 목록
jikjicode doctor                  # 환경/설정/도구 진단
jikjicode session list
jikjicode session export <id> -o recovery.json
jikjicode session import recovery.json
```

주요 옵션:

| 옵션 | 설명 |
|---|---|
| `--config <file>` | 사용할 설정 파일 |
| `-m, --model <name>` | 활성 모델 |
| `-C, --cd <dir>` | 작업 디렉터리 |
| `--compatibility-mode <mode>` | `claude-code`, `openharness`, `native` |
| `-s, --sandbox <mode>` | `read-only`, `workspace-write`, `danger-full-access` |
| `--ask <mode>` | `never`, `on-request`, `untrusted` |
| `--plan` | 읽기 전용 Plan 모드로 시작 |
| `--max-steps <n>` | 최대 agent loop 단계 |
| `--timeout <duration>` | turn별 실행 제한 |
| `--orchestration` | 설정과 관계없이 로컬 오케스트레이션 활성화 |
| `--no-orchestration` | 로컬 오케스트레이션 비활성화 |

로컬 모드는 오케스트레이션이 기본적으로 활성화됩니다.

## TUI 키 바인딩

| 키 | 동작 |
|---|---|
| `Enter` | 입력 전송. 실행 중이면 다음 turn 대기열에 추가 |
| `Alt+Enter` | 실행 중 agent에 즉시 지시, 불가능하면 대기열에 추가 |
| `Ctrl+J` | 입력 줄바꿈 |
| `Up` / `Down` | 입력 커서 이동 또는 입력 기록 탐색 |
| `Ctrl+R` | 세션 전체 입력 기록 검색 |
| `Esc` | 실행 중단. 예약 입력은 입력창으로 회수 |
| `Esc Esc` | 빈 입력창에서 rewind 선택기 |
| `Ctrl+A` | 대기 상태에서 sub-agent 목록과 상세 정보 열기 |
| `Ctrl+O` | 도구 입력과 전체 출력을 포함해 transcript 다시 렌더 |
| `Ctrl+G` | `$EDITOR`에서 입력 편집 |
| `Shift+Tab` | 승인 정책 순환 |
| `PgUp` / `PgDn` | transcript 스크롤 |
| `/` | 명령 palette |
| `@` | 작업공간 파일 자동 완성 |
| `?` | 단축키 도움말 |
| `Ctrl+C` / `Ctrl+D` | 실행 중이면 중단, 대기 중이면 종료 |

Jikjicode는 터미널 mouse reporting을 활성화하지 않으므로 일반적인 마우스 드래그
선택과 터미널의 복사 기능을 사용할 수 있습니다.

실행 중인 Agent 또는 task graph가 있으면 TUI의 fleet 영역에 전체/실행/완료 수와
각 Agent 상태가 표시됩니다. `/agents`로 현재 세션의 실행 중·완료·실패 Agent를
조회하고 `/agents <id>`로 상세 상태를 확인할 수 있습니다.

## 주요 Slash 명령

| 분류 | 명령 |
|---|---|
| 도움/상태 | `/help`, `/status`, `/usage`, `/context`, `/version`, `/mode`, `/tools` |
| 설정 | `/config`, `/model`, `/effort`, `/tool` |
| Plan/작업 | `/plan`, `/todo`, `/taskgraph`, `/goal` |
| 오케스트레이션 | `/autoresearch`, `/ensemble`, `/oracle` |
| 세션 | `/new`, `/fork`, `/save`, `/rename`, `/sessions`, `/resume`, `/search` |
| 컨텍스트 | `/rewind`, `/compact`, `/clear` |
| Agent/실행 관리 | `/agents`, `/ps`, `/stop`, `/schedule` |
| 프로젝트 | `/diff`, `/init`, `/soul` |
| 출력/종료 | `/copy`, `/export`, `/quit` |

`/plan <요청>`은 읽기 전용 Plan 모드로 들어갑니다. `/plan view`로 확인하고
`/plan execute`로 고정된 revision을 승인한 뒤 실행합니다. `/rewind`는 대화
컨텍스트를 분기하지만 작업공간 파일을 되돌리지는 않습니다. `/compact`는 완료된
turn을 보관한 다음 provider에 전달되는 대화를 압축합니다.

`/usage`는 캐시된 상태를 즉시 표시하고 provider 사용 제한을 비동기로 갱신합니다.
공식 endpoint와 현재 인증 방식이 지원하면 5시간/주간 제한을 표시하며, 지원하지
않으면 unavailable 사유를 보여줍니다. Provider 이름에는 `OAuth` 또는 `API key`와
같이 현재 모델 경로가 선택한 인증 방식이 함께 표시되며 credential 값 자체는
출력하지 않습니다.

로컬 Agent와 task graph 상태는 Galley SQLite에 영속되어 재시작 및 `/resume` 후에도
복원됩니다. 다만 `session export`는 Galley의 오케스트레이션 행과 task graph를
포함하지 않습니다. 다른 머신으로 함께 옮기려면 동일한 Galley 저장소를 별도로
이전하거나 공유 Galley DSN을 사용해야 합니다.

## 라이선스와 대응 소스

이 스냅샷은 별도의 서면 Apache-2.0 허가를 받은 수령인이 아니라면
GPL-3.0-only 조건으로 배포됩니다. 전체 조건은 [`LICENSE`](./LICENSE)와
[`LICENSING.md`](./LICENSING.md)를 확인하십시오. 별도 Apache-2.0 허가는
[Jikji Labs 연락처](https://jikji-labs.com/contact.html)를 통해 요청할 수 있습니다.

이 바이너리의 정확한 대응 소스는 Jikji commit
[`73db2dbb19f711deb904a7f259cdbf271054e479`](https://github.com/jikji-labs/jikji/commit/73db2dbb19f711deb904a7f259cdbf271054e479)
입니다. 현재 원본 저장소 접근 권한이 없는 수령인을 위한 GPL 대응 소스 제공 방법과
유효 기간은 [`SOURCE_OFFER.md`](./SOURCE_OFFER.md)에 명시되어 있습니다.

## 문제 보고

이 스냅샷은 내부 사용(dogfooding)과 호환성 검증을 위한 pre-test입니다. 문제를
보고할 때 다음 정보를 포함하면 재현에 도움이 됩니다.

```text
/version
/status
/context
재현 단계와 기대 동작
```

로그, 설정, session export를 첨부하기 전에 API 키, OAuth 토큰, 개인정보와 사내
경로를 제거하십시오. `session export`는 도구 입출력과 복구 상태를 포함하는 민감한
파일이며 기본 권한은 소유자 전용입니다.
