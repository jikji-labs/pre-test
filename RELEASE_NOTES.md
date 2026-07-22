# 2026-07-22-snapshot-p1

이 릴리스는 정식 제품 릴리스가 아닌 Linux amd64 사전 테스트 스냅샷입니다.
대응 소스 commit은 `a03bf8cd8bc96517f700de731ca7ab8508e3ca26`입니다.

## Temporal PII redaction

- `/config redaction off`가 현재 실행에서 생성된 `Agent`, `taskgraph` 및 중첩
  에이전트에 상속됩니다.
- 원격 Jikjicode에서 Jikji 서버를 사용하는 경우에도 같은 정책이 Press, Forme,
  durable dispatch 경계를 통과합니다.
- wait 및 approval park 이후 `/resume`에서도 유효한 temporal capability를
  복원합니다.
- 원격 bypass는 `operator:write` 권한이 필요하며 최대 8시간으로 제한됩니다.
- 모델은 내부 capability, tenant/session/run 출처 또는 만료 시간을 지정할 수
  없습니다.
- 새 세션과 새 프로세스는 redaction ON으로 시작합니다. 이미 서버가 승인한
  cursor-zero 실행의 정확한 재전송 정보만 해당 실행에 한정해 보존합니다.
- `jikjictl run --redaction on|off`와 OpenAPI 실행 계약을 추가했습니다.

## Provider tool replay

- Codex Responses에서 tool name fragment가 잘못 조립된 경우, 사용자에게 출력되기
  전에는 해당 응답을 폐기하고 한 번의 bounded rewind를 수행합니다.
- tool name을 추측하거나 자동 변환하지 않으며, 이미 출력이 공개된 경우에는
  중복 실행을 막기 위해 fail closed로 처리합니다.

## 구조 및 검증

- RemoteRunner, Press, Forme의 redaction 정책을 전용 모듈로 분리하여 대형 파일
  정책을 준수합니다.
- 전체 `make verify`, 관련 race detector, DuckDB 태그 회귀 테스트를 통과했습니다.
- Linux, macOS, Windows 테스트 교차 컴파일과 FreeBSD/OpenBSD portability build를
  검증했습니다.
- 공개 소스 shadow export와 critical coverage 검증을 통과했습니다.

## 포함 파일

- `jikjicode-linux-amd64`: 일반 정적 빌드
- `jikjicode-full-linux-amd64`: DuckDB 포함 확장 빌드
- `jikjicode-darwin-arm64`: macOS Apple Silicon 일반 빌드
- `SHA256SUMS`: 배포 파일 무결성 확인
- `SHA256SUMS.sig`, `release-signing-public.pem`: 배포 체크섬 인증
- `release-metadata.json`: 릴리스 identity와 대응 소스 결속
- `LICENSE`, `LICENSING.md`, `SOURCE_OFFER.md`: 라이선스 및 대응 소스 안내

Full 빌드는 Linux amd64용이며 내장 CPython을 포함하지 않습니다.

## 주의

- 프로덕션 배포용 정식 릴리스가 아닙니다.
- 설정과 로그에 API key, OAuth token 또는 실제 DSN을 기록하지 마십시오.
- 기본 라이선스는 GPL-3.0-only입니다. 대응 소스 요청은 `SOURCE_OFFER.md`를
  따르십시오.
