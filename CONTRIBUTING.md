# Fall Guardian 협업 방법

모든 팀원은 작업 전에 저장소 루트의 `AGENTS.md`를 읽는다.

## 브랜치

통합 브랜치는 `wear-os-port`다. 직접 커밋하지 않고 기능 브랜치에서 Pull Request를 만든다.

```text
feat/wear-sensor-probe
feat/ml-watch-preprocessing
feat/api-fall-events
feat/web-alert-list
fix/int8-calibration
docs/design-freeze
```

작업 시작:

```bash
git switch wear-os-port
git pull --ff-only
git switch -c feat/짧은-작업명
```

## 커밋

커밋은 실행 가능한 작은 단위로 나눈다.

```text
feat(wear): collect accelerometer and gyroscope samples
feat(api): persist fall events in Supabase
fix(ml): align six channel normalization
test(api): cover duplicate SOS requests
docs: update sensor input contract
chore: pin conversion dependencies
```

사용 범주:

- `feat`: 기능 추가
- `fix`: 버그 수정
- `test`: 테스트
- `docs`: 문서
- `chore`: 설정과 의존성
- `refactor`: 동작을 바꾸지 않는 정리

## Pull Request

PR 본문에는 아래 내용만 간단히 적는다.

```markdown
## 변경 목적

## 주요 변경

## 실행한 검증

## 영향 범위와 남은 과제
```

PR 체크리스트:

- [ ] 한 가지 목적만 다룬다
- [ ] 관련 테스트와 정적 검사를 실행했다
- [ ] 비밀정보와 실제 건강 데이터를 포함하지 않았다
- [ ] API·스키마·모델 입력 변경을 문서화했다
- [ ] 오프라인·오류 상황을 확인했다
- [ ] 다른 팀원 한 명이 검토했다

## 작업 충돌 방지

- 작업 시작 전에 담당 이슈와 수정 예정 경로를 공유한다.
- 같은 파일을 동시에 크게 수정하지 않는다.
- 공통 계약은 먼저 작은 문서 또는 타입 PR로 합의한다.
- 충돌 해결 후에는 관련 테스트를 다시 실행한다.
- 다른 사람의 변경을 임의로 되돌리지 않는다.

## 리뷰 기준

리뷰어는 다음 순서로 확인한다.

1. 안전, 개인정보, 인증과 권한
2. 동결된 설계와 API·모델 입력 계약
3. 오프라인 및 실패 처리
4. 테스트와 재현 가능성
5. 코드의 단순성과 읽기 쉬움

사소한 스타일 의견보다 데이터 손실, 중복 SOS, 센서 단위 오류, RLS 누락을 먼저 지적한다.

## 병합

- 승인된 PR만 `wear-os-port`에 병합한다.
- 가능하면 Squash merge를 사용한다.
- 병합 후 기능 브랜치를 삭제한다.
- `wear-os-port`가 검증된 시점에만 릴리스 태그를 만든다.

