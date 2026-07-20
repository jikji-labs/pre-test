# 2026-07-20-snapshot-p1

이 릴리스는 정식 제품 릴리스 전 사용성 및 provider 호환성 검증을 위한
사전 테스트 스냅샷입니다.

## p1 변경 사항

### 실행 복구와 내구성

- provider turn identity가 도구 실행 범위를 벗어나 유효하지 않을 때, 복구 가능한
  경우 모델이 다음 turn에서 스스로 교정할 수 있도록 구조화된 오류 결과를
  전달합니다.
- 등록되지 않은 도구, 잘못된 대기 호출, 실행 전 도구 이름 스트림 조립 오류를
  제한적으로 복구합니다.
- 병렬 도구 묶음의 실행 전 검증을 원자적으로 처리하여 일부 도구만 먼저 실행되는
  상황을 방지합니다.
- 실제 부작용이 시작된 뒤의 실패는 자동 재실행하지 않습니다.

### Agent 상태

- `Agent`, `taskgraph.run`, ensemble 및 orchestrator 경계에서 실행 상태를 자동
  탐색합니다.
- 실행 중인 Agent가 있으면 TUI fleet 영역이 자동으로 나타납니다.
- `/agents`와 `/agents <id>`로 전체 목록과 상세 상태를 확인할 수 있습니다.
- Galley에 저장된 Agent와 task graph 상태를 시작 및 `/resume` 시 다시 구성합니다.

### 진단 추적

- 사용자에게 표시되는 오류마다 `trc_...` 형식의 `trace_id`가 함께 출력됩니다.
- `/search <trace_id>`로 세션 기록에서 해당 오류를 검색할 수 있습니다.
- `/tracepoint [메모]`로 현재 세션, turn, step, 모델 및 컨텍스트 위치를 수동으로
  기록할 수 있습니다.
- 오류 trace와 tracepoint는 세션에 저장되어 `/resume` 후에도 조회할 수 있습니다.

## 포함 파일

- `jikjicode-linux-amd64`: 일반 정적 빌드
- `jikjicode-full-linux-amd64`: DuckDB 포함 확장 정적 빌드
- `SHA256SUMS`: 배포 파일 무결성 확인

Full 빌드는 DuckDB 분석 엔진을 포함하지만 내장 CPython은 포함하지 않습니다.
시스템에 `python3`가 있으면 두 빌드 모두 Python 도구를 외부 프로세스로 실행할 수
있습니다.

## 테스트된 Provider

- Grok OAuth
- GPT / Codex OAuth
- Ollama Pro / Cloud
- GLM (Z.AI) Coding Plan

Provider가 공식 사용 제한 endpoint를 제공하는 경우 `/usage`에서 현재 인증 방식과
5시간/주간 제한을 확인할 수 있습니다. 지원하지 않거나 일시적으로 조회할 수 없으면
추정값을 만들지 않고 unavailable 사유를 표시합니다.

## 임시 보호 기능 우회

권한 문제로 OS sandbox를 임시 해제해야 할 때는 `/config`에서 `sandbox_flip`을
`off`로 설정합니다. IP 주소 등 PII 처리 때문에 모델 전달 데이터가 마스킹되는
경우에는 `redaction`을 `off`로 설정할 수 있습니다.

```text
/config sandbox-flip off session
/config redaction off session
```

두 설정은 현재 실행 동안만 유지되며 `/new`, `/fork` 또는 종료 시 복구됩니다.
권한과 개인정보 보호 기능을 영구적으로 끄는 설정이 아닙니다. 신뢰할 수 있는 작업과
필요한 시간 범위에서만 사용하십시오.

## 주의

- 지원 대상은 Linux amd64입니다.
- 프로덕션 배포용 정식 릴리스가 아닙니다.
- 설정 및 로그에 API 키나 OAuth token을 기록하지 마십시오.
- 기본 라이선스는 GPL-3.0-only입니다. 대응 소스 요청은 저장소의
  `SOURCE_OFFER.md`를 따르십시오.
