# 2026-07-20 Snapshot

이 릴리스는 정식 제품 릴리스 전 사용성 및 provider 호환성 검증을 위한
사전 테스트 스냅샷입니다.

## 포함 파일

- `jikjicode-linux-amd64`: 일반 정적 빌드
- `jikjicode-full-linux-amd64`: DuckDB 포함 확장 정적 빌드
- `SHA256SUMS`: 배포 파일 무결성 확인

## 주의

- 지원 대상은 Linux amd64입니다.
- Full 빌드는 이번 스냅샷에서 DuckDB를 포함하며 내장 CPython은 포함하지 않습니다.
- Python 도구는 시스템에 `python3`가 설치된 경우 외부 프로세스로 실행합니다.
- 프로덕션 배포용 정식 릴리스가 아닙니다.
- 설정 및 로그에 API 키나 OAuth token을 기록하지 마십시오.
- 기본 라이선스는 GPL-3.0-only입니다. 대응 소스 요청은 저장소의
  `SOURCE_OFFER.md`를 따르십시오.
