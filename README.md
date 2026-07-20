# Jikjicode 사전 테스트 스냅샷

이 저장소는 Jikjicode의 사전 공개 테스트 바이너리와 실행 안내를 제공합니다.
현재 배포본은 **정식 릴리스가 아니며 프로덕션 사용을 보장하지 않습니다.**
소스 코드는 이 저장소에 포함하지 않습니다.

## 2026-07-20-snapshot-p5

지원 환경은 Linux x86-64(amd64)입니다.

| 파일 | 구성 |
|---|---|
| `jikjicode-linux-amd64` | 일반 정적 바이너리 |
| `jikjicode-full-linux-amd64` | DuckDB 분석 엔진을 포함한 확장 정적 바이너리 |
| `SHA256SUMS` | 배포 파일 SHA-256 체크섬 |

Full 바이너리는 이번 스냅샷에서 DuckDB를 포함합니다. 내장 CPython은 아직 포함하지
않습니다. Python 도구는 시스템에 `python3`가 설치된 경우 두 바이너리 모두 외부
프로세스 방식으로 사용할 수 있습니다.

### 테스트된 Provider

이 스냅샷에서 실제 인증과 대화 경로를 확인한 provider는 다음과 같습니다.

| Provider | 검증된 인증/구독 경로 |
|---|---|
| Grok | OAuth |
| GPT / Codex | OAuth |
| Ollama | Ollama Pro / Cloud |
| GLM (Z.AI) | Coding Plan |

이 목록은 현재 테스트를 완료한 경로이며 전체 provider 호환성 목록을 의미하지
않습니다. 활성 모델과 인증 방식, provider가 제공하는 사용 제한 정보는 TUI의
`/usage`에서 확인할 수 있습니다.

### p5 주요 변경

- 심볼릭 링크로 연결된 작업 디렉터리를 시작 시 안전하게 실제 디렉터리로
  고정합니다. private tool output buffer의 링크 우회 방지는 유지하면서
  `~/ontology -> /docker/ontology` 같은 구성이 정상적으로 시작됩니다.
- 비 Git 디렉터리에서도 대화형 `jikjicode`를 시작할 수 있습니다. positional
  goal과 `jikjicode exec`의 headless 실행은 기존 Git 저장소 검사를 유지합니다.
- MCP server 연결과 해제를 직렬화하고 `/config`의 enable/disable 동작을
  비동기로 처리하여 TUI 정지와 중복 연결을 방지합니다.
- ontology graph/provenance 상태를 원자적으로 갱신하고 `/status`에 graph,
  tuple, 메모리 및 embedding cache 사용량을 일관된 snapshot으로 표시합니다.
- Full 바이너리는 tenant allowlist와 read-only/read-write 모드를 갖춘 DuckDB
  PostgreSQL/MySQL connector를 지원합니다. DSN 값은 YAML이 아니라 환경 변수로만
  전달합니다.
- `/duckdb`에서 connector 상태를 확인할 수 있으며, `/config sqlguard`의
  선택적 heuristic filter는 로컬 기본값이 `off`입니다. SELECT-only 검증,
  tenant scoping, stacked statement 차단과 DuckDB sandbox는 항상 유지됩니다.
- 터미널 실행은 24시간마다 GitHub prerelease를 비동기로 확인하고 새 버전만
  알립니다. 자동 설치하지 않으며 `jikjicode update`를 명시해야 합니다.
- 업데이트는 현재 standard/full 빌드 종류와 OS/architecture가 일치하는 자산만
  선택하고, 크기 제한과 `SHA256SUMS` 검증 후 같은 디렉터리에서 원자 교체합니다.

- provider turn identity와 도구 실행 경계가 어긋났을 때 실행을 중단하는 대신,
  안전하게 수정 가능한 오류를 모델에 돌려주어 다음 동작에서 스스로 교정할 수
  있도록 복구 루프를 보강했습니다.
- 잘못 조립된 도구 이름과 provider 대기 호출을 실행 전에 감지하고 제한적으로
  재시도합니다. 병렬 도구 묶음은 하나라도 승인되지 않으면 일부만 실행되지
  않도록 원자성을 유지합니다.
- 실행 중인 Agent와 task graph를 Galley의 영속 상태에서 자동 탐색하여 TUI fleet
  영역에 표시합니다. 재시작과 `/resume` 뒤에도 상태를 다시 구성합니다.
- 사용자에게 표시되는 오류에 `trace_id`를 부여합니다. `/search <trace_id>`로
  관련 기록을 찾고, `/tracepoint [메모]`로 현재 세션 위치를 수동 기록할 수
  있습니다.

자동 복구는 실행 전이며 부작용이 없다고 확인되는 경계에서만 동작합니다. 이미
실행된 변경 작업이나 승인·보안 경계를 임의로 다시 실행하지 않습니다.

## 설치

릴리스 페이지에서 원하는 바이너리와 `SHA256SUMS`를 받은 뒤 무결성을 확인합니다.

```bash
sha256sum -c SHA256SUMS --ignore-missing
install -m 0755 jikjicode-linux-amd64 ~/.local/bin/jikjicode
jikjicode version
```

Full 바이너리를 설치하려면 두 번째 줄의 파일명을
`jikjicode-full-linux-amd64`로 바꿉니다.

심볼릭 링크 또는 비 Git 데이터 디렉터리에서도 대화형 모드는 그대로 시작합니다.
자동화된 one-shot 실행은 Git 저장소가 아니면 명시적으로 승인해야 합니다.

```bash
jikjicode                              # 비 Git 디렉터리의 대화형 세션
jikjicode --skip-git-repo-check "점검" # 비 Git 디렉터리의 one-shot 실행
```

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

### 임시 보호 기능 우회

파일 권한이나 host 명령 실행이 OS sandbox에 막혀 작업을 진행할 수 없을 때는
`/config`의 `sandbox_flip`을 `off`로 설정할 수 있습니다. 명령으로 직접 적용하려면
다음과 같이 실행합니다.

```text
/config sandbox-flip off session
```

활성화되면 `/config`에는 다음과 비슷하게 표시됩니다.

```text
redaction             ‹ OFF — bypassed (session (until /new, /fork, or exit)) ›
shell_sandbox         ‹ bubblewrap → none (flipped: session (until /new, /fork, or exit)) ›
sandbox_flip          ‹ OFF — sandbox flipped off (session (until /new, /fork, or exit)) ›
```

`sandbox_flip`은 `shell.run`과 새로 시작하는 PTY의 OS 격리를 해제하므로, 명령이
호스트의 실제 HOME, `/run`, `docker.sock` 등에 접근할 수 있습니다. 이미 실행 중인
PTY의 격리 상태는 바뀌지 않습니다. Docker나 host daemon 접근만 필요하다면 sandbox를
완전히 끄기 전에 `/config shell-sandbox native` 사용을 우선 검토하십시오.

IP 주소 등 PII 탐지로 인해 모델에 전달되어야 할 도구 입력이나 출력이 마스킹되는
경우에는 `/config`의 `redaction`을 `off`로 설정할 수 있습니다.

```text
/config redaction off session
```

이 설정은 모델과 provider로 전달되는 로컬 도구 입출력의 PII 마스킹을 임시로
우회합니다. 화면의 영속 증거, audit log와 spill 데이터의 redaction은 계속
적용됩니다.

두 설정 모두 현재 세션에서만 유효하며 설정 파일에 영구 저장되지 않습니다.
`/new`, `/fork`, 프로그램 종료 또는 지정한 범위가 끝나면 자동으로 원래 보호
설정으로 복귀합니다. 즉, 문제 해결을 위해 일부 보호·권한 경계를 임시로 우회하는
기능이지 권한 시스템을 영구적으로 끄는 설정이 아닙니다. 즉시 복구하려면 다음을
사용합니다.

```text
/config sandbox-flip on
/config redaction on
```

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
jikjicode update --check
jikjicode update
```

업데이트 확인은 오프라인 기동을 막지 않으며 실패 시 조용히 다음 기회로
넘어갑니다. air-gapped 또는 중앙 배포 환경에서는
`JIKJICODE_NO_UPDATE_CHECK=1`로 백그라운드 확인을 끌 수 있습니다. 명시적인
`jikjicode update` 명령은 계속 사용할 수 있습니다.

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
| 세션 | `/new`, `/fork`, `/save`, `/rename`, `/sessions`, `/resume`, `/search`, `/tracepoint` |
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

오류가 표시되면 함께 출력되는 `trace_id`를 문제 보고에 포함하십시오. 같은 세션의
기록은 `/search trc_...`로 찾을 수 있으며 `/resume` 후에도 유지됩니다. 오류가
발생하지 않았지만 특정 실행 지점을 남겨야 할 때는 다음과 같이 기록합니다.

```text
/tracepoint provider 응답 대기 직전
```

Tracepoint에는 세션, turn, step, 모델, 컨텍스트 사용량과 가능한 경우 원격 run ID가
저장됩니다. API 키, OAuth 토큰과 도구 입력 전문은 trace ID 자체에 포함되지 않습니다.

## 라이선스와 대응 소스

이 스냅샷은 별도의 서면 Apache-2.0 허가를 받은 수령인이 아니라면
GPL-3.0-only 조건으로 배포됩니다. 전체 조건은 [`LICENSE`](./LICENSE)와
[`LICENSING.md`](./LICENSING.md)를 확인하십시오. 별도 Apache-2.0 허가는
[Jikji Labs 연락처](https://jikji-labs.com/contact.html)를 통해 요청할 수 있습니다.

각 바이너리에 내장된 정확한 Jikji 소스 commit은 `jikjicode version`에서 확인할 수
있습니다. 현재 원본 저장소 접근 권한이 없는 수령인을 위한 GPL 대응 소스 제공
방법과 유효 기간은 [`SOURCE_OFFER.md`](./SOURCE_OFFER.md)에 명시되어 있습니다.

## 문제 보고

이 스냅샷은 내부 사용(dogfooding)과 호환성 검증을 위한 pre-test입니다. 문제를
보고할 때 다음 정보를 포함하면 재현에 도움이 됩니다.

```text
/version
/status
/context
오류에 표시된 trace_id
재현 단계와 기대 동작
```

로그, 설정, session export를 첨부하기 전에 API 키, OAuth 토큰, 개인정보와 사내
경로를 제거하십시오. `session export`는 도구 입출력과 복구 상태를 포함하는 민감한
파일이며 기본 권한은 소유자 전용입니다.
