# 왕가네 장애 시험 (TEST_3)

왕가네 플랫폼의 운영 알림(스파이크 감지 → 자동 중지 · AI 판단 → Discord)을 시험하려고 **일부러 장애를 내는** 작은 Node 서버입니다. 설치할 의존성이 없습니다.

| 경로 | 응답 |
| --- | --- |
| `GET /` | 소개 화면 |
| `GET /api/health` | 정상이면 `200 {"ok":true}`, 장애 중이면 `503 {"ok":false}` |
| `GET /api/info` | `{ uptimeSeconds, chaos }` |

## 배포 설정

- Dockerfile: `Dockerfile` (루트)
- 실행 방식: 서버(server), 포트 `8080`
- 공개 경로: `/`
- 동작 확인: http `/api/health`
- 비밀값: `CHAOS` (아래 시나리오), 선택 `CHAOS_AFTER_SECONDS` (기본 `180`)

## 시나리오

장애는 시작하고 `CHAOS_AFTER_SECONDS`초 뒤에 납니다. 배포 확인과 공개 주소 전환을 정상으로 통과한 뒤에 나게 하려는 것입니다.

| `CHAOS` | 하는 일 | 기대하는 플랫폼 동작 |
| --- | --- | --- |
| (비움) | 정상 서버 | 알림 없음 |
| `crash` | 180초 뒤 종료 코드 1로 끝남 → 재시작 반복 | 약 15~25분 뒤 `KubePodCrashLooping` → 진행 중 배포가 없으므로 **자동 중지**(AI 부르지 않음) → 운영자 · 학생 Discord |
| `unready` | 180초 뒤 `/api/health`가 503 → Pod가 준비 안 됨(재시작은 없음) | 약 25분 뒤 `KubeDeploymentReplicasMismatch` → **AI 판단**(원인 · 위험 · 권고) → 운영자 Discord · 학생 Discord, 운영자 화면에서 중지 / 무시 |

로그에는 `chaos: ... test fault`가 남습니다. 시험용 장애라는 것을 숨기지 않습니다.
