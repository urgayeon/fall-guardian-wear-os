# Fall Guardian 개발 규약

이 문서는 사람과 코딩 에이전트가 같은 설계를 기준으로 작업하기 위한 저장소 최상위 규칙이다.
충돌이 있으면 보안과 개인정보 보호, 이 문서, 하위 디렉터리의 규칙 순서로 적용한다.

## 목표

Galaxy Watch 8에서 손목 가속도와 자이로를 수집하고 워치 내부 LiteRT 모델로 낙상을 감지한다.
워치는 현장 경고와 오프라인 SOS 재전송을 담당한다. 회사 제공 PC의 FastAPI 서버는 요청을
검증하고 Supabase에 저장한다. 관리자 웹은 Supabase의 인증과 권한 정책을 적용해 사건을 보여준다.

## 동결된 설계

- 기준 저장소: `urgayeon/fall-guardian-wear-os`
- 통합 브랜치: `wear-os-port`
- 원본 기준 태그: `upstream-5879f92`
- 워치 앱: Kotlin, Jetpack Compose for Wear OS, LiteRT
- 움직임 센서: Android `SensorManager`
- 건강 센서: Samsung Health Sensor SDK
- 서버: 회사 제공 PC의 Docker와 FastAPI
- 데이터베이스: Supabase PostgreSQL, Auth, RLS, Realtime
- 관리자 웹: React와 TypeScript
- 공개 배포: 개발 완료 후 Samsung Health Sensor SDK 파트너 승인과 Galaxy Store 심사

다음 변경은 팀 합의와 설계 문서 버전 갱신 없이 하지 않는다.

- 모델 입력 `[1, 125, 6]`
- 채널 순서 `accel_x, accel_y, accel_z, gyro_x, gyro_y, gyro_z`
- 워치 내부 추론 원칙
- 회사 PC와 Supabase의 책임 경계
- 낙상 이벤트 상태와 API 계약
- 오프라인 SOS의 재전송 및 중복 방지 원칙

## 저장소 구조

```text
ml/          모델 변환, 학습, 평가
wear/        Galaxy Watch 8 Wear OS 앱
backend/     FastAPI 서버
dashboard/   관리자 웹
mobile/      기존 Flutter 모바일 앱
docs/        설계, API, 검증 결과
```

없는 디렉터리는 해당 기능을 처음 구현하는 PR에서 최소 구조로 만든다.

## 구현 원칙

### Wear OS

- 가속도와 자이로 이벤트의 실제 타임스탬프를 보존한다.
- 센서 도착 순서를 모델 입력 순서로 간주하지 않는다. 공통 시간축으로 정렬하고 50Hz로 리샘플링한다.
- 단위, 축 방향, 정규화 통계를 코드와 문서에 함께 기록한다.
- 센서나 건강 데이터가 없을 때 정상값을 만들어 넣지 않는다. `unavailable` 상태를 명시한다.
- 인터넷이 없어도 감지, 경고, 취소 화면은 동작해야 한다.
- 미전송 SOS는 로컬에 보존하고 재부팅 후에도 재전송해야 한다.

### 모델

- 기준 입력은 125개 시점의 6축 데이터다.
- 모델 파일과 정규화 통계는 버전과 SHA-256을 함께 기록한다.
- INT8 판정 일치율이 FP32 대비 99% 미만이면 재보정하거나 FP32 TFLite를 사용한다.
- 테스트 데이터로 임계값을 조정하지 않는다. 검증용 데이터와 최종 평가 데이터를 분리한다.
- WEDA-FALL 사용 허가가 확인되기 전에는 원본 데이터와 파생 가중치를 공개 배포하지 않는다.

### 서버와 Supabase

- 워치가 Supabase의 관리자 권한으로 직접 쓰지 않게 한다. 낙상 이벤트는 FastAPI를 통한다.
- 모든 이벤트 생성 요청에는 `idempotency_key`를 사용한다.
- Supabase secret key는 회사 PC의 환경변수에만 저장한다.
- 브라우저와 앱에는 publishable key만 사용하고 모든 공개 테이블에 RLS를 적용한다.
- 기존 로컬 PostgreSQL과 Redis 구성은 참고용이다. 운영 데이터 저장소는 Supabase다.
- 로그에 토큰, 건강 원시 데이터, 개인정보를 기록하지 않는다.

### 관리자 웹

- 인증된 관리자에게 허용된 대상만 표시한다.
- 마지막 통신 시각, 미동기화, 서버 장애를 정상 상태처럼 숨기지 않는다.
- 알림 확인, 연락, 종결 같은 변경은 담당자와 시각을 이력으로 남긴다.

## 보안과 데이터

절대로 커밋하지 않는다.

- `.env`, API 키, GitHub 토큰, Supabase secret key
- Android 릴리스 키와 비밀번호
- WEDA-FALL 원본 데이터
- 실제 사용자 건강 데이터와 개인정보
- 승인 전 파생 모델 가중치

예제 환경변수는 값이 없는 `.env.example`로 제공한다. 테스트 데이터는 합성값이나 비식별 샘플만 쓴다.

## 작업 방법

1. `wear-os-port` 최신 상태에서 작업 브랜치를 만든다.
2. 한 브랜치에서는 한 가지 목적만 처리한다.
3. 기존 코드와 패턴을 먼저 찾고 새 의존성과 추상화는 꼭 필요한 경우에만 추가한다.
4. 변경한 구성요소의 최소 테스트와 정적 검사를 실행한다.
5. API, 데이터 스키마, 모델 입력이 바뀌면 같은 PR에서 문서를 갱신한다.
6. PR은 최소 한 명의 팀원 검토를 받은 뒤 병합한다.

## 기본 검증 명령

### ML

```bash
cd ml
uv run pytest
uv run ruff check .
python scripts/validate_tflite.py
```

### Backend

```bash
cd backend
uv run pytest
uv run ruff check .
```

### Flutter mobile

```bash
cd mobile
flutter analyze
flutter test
```

### Wear OS

```bash
cd wear
./gradlew test lint
```

Windows에서는 Wear OS Gradle 명령에 `gradlew.bat`을 사용한다. 존재하지 않는 구성요소의 명령은
해당 구성요소를 만드는 PR에서 실제 실행 가능한 명령으로 교체한다.

## 완료 기준

작업은 다음 조건을 모두 만족해야 완료다.

- 요구사항이 실제 코드로 동작한다.
- 관련 테스트와 정적 검사가 통과한다.
- 비밀정보와 데이터가 커밋되지 않았다.
- 실패와 오프라인 상태를 처리한다.
- API, 스키마, 모델 입력 변경이 문서화됐다.
- 재현에 필요한 명령과 검증 결과가 PR에 기록됐다.

## 금지 사항

- `wear-os-port`에 직접 푸시하지 않는다.
- 검증 없이 임계값이나 정규화 값을 변경하지 않는다.
- 센서 데이터 누락을 0이나 정상값으로 조용히 대체하지 않는다.
- 워치에서 서버 추론을 전제로 구현하지 않는다.
- 테스트를 통과시키기 위해 안전·권한 검사를 제거하지 않는다.
- 필요하지 않은 프레임워크, 서비스, 메시지 큐를 미리 추가하지 않는다.

