# 점심 선택 시스템 · 점심/여행 자동 결정 V8.4.3

한국·일본의 지역·상권을 선택하고 등록된 음식점·관광지·카페·숙박으로 점심과 여행 일정을 구성하는 시스템입니다.

- **현재 코드 버전:** V8.4.3 — 여행코스 화면 및 지도 길찾기 수정
- **최종 추가 요구사항:** V8.5.1 — 관리자 「AI 지역 뉴스」 메뉴, Fact Sheet 입력·검증, `gpt-4.1-mini` 기사 초안 생성
- **문서 기준일:** 2026-10-19

> 현재 구현과 후속 개발 목표를 구분합니다. 이 저장소에는 V8.5.1 뉴스 기능이 아직 없으며, 이 문서 변경은 뉴스 구현이나 MySQL/OpenAI 통합 테스트 완료를 의미하지 않습니다.

## 현재 기능과 범위

| 구분 | 저장소에서 확인한 기능 |
| --- | --- |
| 지역 선택 | 국가 → 시·도 → 시·군·구 → 읍·면·동 → 상권 |
| 사용자 | 회원가입·로그인, 개인 조건, 점심 추천, 회원 방문/거부 피드백과 이력 |
| 추천 | 예산·평점·선호/기피 음식·최근 회원 이력을 이용한 규칙 기반 점수 |
| 관리자 | 지역·상권, 음식점·카페·관광지·숙박 등록/조회, 지역 연결 진단 |
| 여행 | 1~7일, 날짜별 관광 → 점심 → 카페 → 관광 → 저녁 → 숙박 |
| 장소 분산 | 종류별 장소를 순환 배분하여 데이터가 충분하면 중복 회피 |
| 여행 시연 데이터 | 종로구 음식점·카페·숙박·관광지 각 14곳, 총 56곳 추가 스크립트 |
| 지도 | 음식점 검색, 선택 음식점/직접 입력 장소/여행 단계별 Google 지도 길찾기 |
| 날씨 | OpenWeather 키 설정 시 현재 날씨 조회 |
| 구간 이동시간 | 실제 장소의 좌표와 OSRM 자동차 경로 응답이 확인된 구간만 계산 |
| Google Places 수집 | 자동 동기화는 현재 HTTP 501로 비활성화 |
| AI 지역 뉴스 | V8.5.1 개발 목표, 현재 메뉴·뉴스 API·OpenAI 연동 코드 없음 |

56곳은 **가상 시연 장소**입니다. 실제 매장·호텔로 안내하지 않으며, 가상 장소의 지도는 해당 지역까지만 연결합니다. 실제 장소의 이름·주소 또는 확인된 좌표를 등록해야 개별 장소 길찾기가 가능합니다.

현재 여행 엔진은 종류별 장소 분산과 경로 확인용입니다. 실제 영업시간·예약·날씨·예산·교통을 동시에 최적화하는 완성형 여행 예약 시스템으로 검증된 상태는 아닙니다.

## V8.4.3 지도와 화면 변경

- 여행 일정은 `itinerary-*` 전용 컴포넌트로 출력하여 기존 CSS 충돌을 피하도록 구성했습니다.
- CSS/JS 주소의 `?v=843` 버전 표시로 이전 캐시 사용을 줄입니다.
- 지도 검색·길찾기는 Google 지도 URL로 연결합니다. 관리자 로그인·장소 저장·Google API 키 없이 사용할 수 있습니다.
- 직접 길찾기에는 장소명 또는 주소를 입력합니다. 좌표를 수동 입력할 필요가 없습니다.
- 출발 기준은 현재 위치 또는 선택 지역으로 지정합니다.
- 지도 링크와 OSRM 구간 이동시간 계산은 별도 기능입니다.

상세 변경 내역: [V8.4.3](README_V8.4.3.md), [V8.4.2](README_V8.4.2.md)

## 기술 구성과 주요 파일

| 항목 | 구성 |
| --- | --- |
| 서버 | Node.js, Express 5 |
| 데이터베이스 | MySQL 8.x, mysql2 |
| 화면 | HTML / CSS / JavaScript |
| 인증 | express-session, bcryptjs |
| 외부 서비스 | OpenWeather, OSRM, Google 지도 링크 |
| 뉴스 생성 목표 | OpenAI API, `gpt-4.1-mini` |

`package.json`에 명시된 Node.js 최소 버전은 18.18입니다.

| 파일 | 역할 |
| --- | --- |
| [server.js](server.js) | 서버·세션·정적 화면·상태 확인 |
| [routes/api.js](routes/api.js) | 지역·회원·관리자·추천·여행 API |
| [public/index.html](public/index.html) | 사용자 화면 |
| [public/admin.html](public/admin.html) | 관리자 화면 |
| [public/map-links.js](public/map-links.js) | 지도 검색·길찾기 URL 생성 |
| [lib/travel-plan.js](lib/travel-plan.js) | 날짜별 장소 분산 및 확인된 이동시간 합산 |
| [ERD-V8.4.md](ERD-V8.4.md) | 지역·장소·회원·이관 관계 |
| [scripts](scripts/) | 백업·설치·시드·이관·검증 |
| [.env.example](.env.example) | 설정 예제 |

## Windows PowerShell 실행

### 저장소 내려받기

저장소 폴더와 그 안의 실행 폴더 이름이 같습니다. `package.json`이 있는 **안쪽 폴더**에서 명령을 실행합니다.

```powershell
git clone https://github.com/choisunghwan1004-dev/lunch-decision-v8.4.3-map-layout-fixed.git
cd lunch-decision-v8.4.3-map-layout-fixed
cd lunch-decision-v8.4.3-map-layout-fixed
npm install
```

### 처음 설치하는 경우

`.env`가 없을 때만 예제를 복사합니다.

```powershell
Copy-Item .env.example .env
notepad .env
```

MySQL 접속 정보, 관리자 비밀번호, `SESSION_SECRET`을 실제 값으로 설정합니다. 현재 자동 생성 안전 장치상 신규 DB 이름은 `V83_DB_NAME=lunch_decision_v83`이며, 기존 DB `OLD_DB_NAME=lunch_decision`과 달라야 합니다.

기존 데이터가 있다면 먼저 `npm run backup`의 성공 여부와 결과 파일을 확인합니다. 이후 신규 DB 준비가 필요한 경우에만 다음을 실행합니다.

```powershell
npm run setup
npm run seed:demo
npm run verify
npm run verify:travel
npm start
```

`seed:demo`에는 56개 여행 샘플 등록도 포함됩니다. 샘플은 실제 영업 정보가 아닙니다. SQL 파일을 기존 DB에 직접 실행하지 않습니다.

### V8.4.1 / V8.4.2에서 업데이트하는 경우

기존 서버를 `Ctrl+C`로 종료하고 별도 폴더에 새 코드를 준비합니다. 기존 `.env`를 실행 폴더에 복사합니다. 이미 정상인 DB를 초기화하거나 이관을 반복할 필요는 없습니다.

```powershell
npm install
npm run backup
npm run doctor
npm run test:ui
npm start
```

여행 샘플이 필요한 경우 백업 후 별도로 실행합니다.

```powershell
npm run seed:travel
npm run verify:travel
```

`종로구 샘플 상권 없음` 오류가 나오면 `npm run seed:demo`로 기본 지역 경로를 준비합니다.

### 접속 주소

- 사용자: http://127.0.0.1:3100/
- 관리자: http://127.0.0.1:3100/admin
- 상태 확인: http://127.0.0.1:3100/health

기본 포트 3100 기준이며 `.env`의 `PORT`에 맞춥니다. 이전 화면이 보이면 `Ctrl+Shift+R`로 새로고침합니다.

## DB 흐름과 기존 데이터 보존

주요 지역 연결은 `countries → provinces → districts → admin_towns → commercial_areas → places`입니다. 법정구역은 `legal_units`와 `town_legal_links`로 연결합니다.

회원은 `users`, 조건은 `user_preferences`, 추천·방문·거부 기록은 `lunch_history`로 관리합니다. `migration_mappings`는 기존 DB와 신규 DB 사이의 ID 대응을 저장합니다.

- 기존 `lunch_decision`은 백업·이관 시 읽기 용도로 사용합니다.
- 신규 `lunch_decision_v83`에 스키마 변경과 이관 데이터를 적용합니다.
- 부모 지역이 불명확한 항목은 보류하며 다른 읍·면·동으로 임의 연결하지 않습니다.
- 실제 이관은 미리보기 보고서를 검토한 뒤 실행하며, 오류 시 신규 DB 트랜잭션을 롤백합니다.
- 상세 백업·이관·복원 방법은 아래의 기존 운영 안내를 보존했습니다.

## V8.5.1 최종 추가 요구사항 — AI 지역 뉴스

종로구 등 지역 축제·관광 정보를 대상으로, **확인된 사실 → 기사 초안 → 관리자 검수 → 후속 승인·게시** 순서로 확장합니다.

| 순서 | 구현할 내용 | 범위 |
| --- | --- | --- |
| 1 | 기존 관리자 화면에 「AI 지역 뉴스」 메뉴 연결 | V8.5.1 |
| 2 | 지역·주제·출처와 Fact Sheet 입력·저장·조회 | V8.5.1 |
| 3 | 필수 항목과 사실별 출처 연결 검증, 오류 표시 | V8.5.1 |
| 4 | 검증을 통과한 사실만 `gpt-4.1-mini`에 전달하여 초안 생성 | V8.5.1 |
| 5 | 초안 문장과 근거 비교, 관리자 검수 기록 | V8.5.1 |
| 6 | 관리자 최종 승인 기능 연결 | 후속 단계 |
| 7 | 승인된 뉴스만 사용자 화면에 공개 | 후속 단계 |

**초안 생성 직후 자동 게시하지 않습니다.** 뉴스용 테이블·필드·API 경로는 구현 시 확정해야 하며, 현재 저장소에 생성되어 있다는 뜻은 아닙니다.

### Fact Sheet 설계 기준

- 지역·주제, 행사/관광지명, 기간·장소, 확인된 수치와 단위를 구조화합니다.
- 출처명·URL·발행일·확인일을 기록하고 사실별로 근거를 연결합니다.
- 누락되거나 검증되지 않은 사실은 생성 대상에서 제외하거나 생성을 차단합니다.
- 확인되지 않은 원인·인터뷰·통계·가격·일정을 생성하지 않도록 합니다.
- 초안의 근거와 수정·검수 기록을 보관합니다.
- API 키 미설정·인증 오류·한도 초과·타임아웃 시 오류를 표시하고 입력 내용을 유지합니다.

### 완료 조건과 작업 관리

- [ ] 기존 UI·스타일·상태·세부 기능을 보존하며 뉴스 메뉴 추가
- [ ] 뉴스 MySQL 스키마와 지역 연결 정의
- [ ] Fact Sheet CRUD 및 검증 구현
- [ ] 검증된 사실만 사용하는 OpenAI 초안 생성 연결
- [ ] 근거 비교와 관리자 검수 기록 저장
- [ ] 실제 MySQL 테이블 생성·저장·조회 통합 테스트
- [ ] 실제 OpenAI API 호출 통합 테스트
- [ ] 후속 관리자 승인·사용자 게시의 접근 권한 검증
- [ ] Notion 「AI사업계획서」 하위에 작업 내용·ERD·흐름·검증 결과 기록

현재 `.env.example`에는 뉴스용 OpenAI 키 설정이 없습니다. 뉴스 모듈 도입 시 환경변수와 서버 호출을 연결해야 합니다. 이 README 변경은 Notion 기록이나 뉴스 코드 구현을 수행하지 않습니다.

## 검증 명령과 확인 방법

실행 폴더에서 다음 명령을 사용합니다.

```powershell
npm run check
npm run test:ui
npm run doctor
npm run verify
npm run verify:travel
npm run test:api
```

- `check`: 주요 JavaScript 문법 검사
- `test:ui`: 지도·지역·여행 관련 Node 테스트. 실제 브라우저 조작과는 별도
- `test:map`: 지도 URL 전용 테스트만 별도로 실행할 때 사용
- `doctor` / `verify`: DB 접속 및 FK·지역 선택 연결 진단
- `verify:travel`: DB의 종류별 장소 수와 3일 일정 연결 확인
- `test:api`: HTTP 화면/API와 1·2·3일의 6·12·18단계 응답 검사

API 테스트는 서버가 없으면 임시 서버를 시작하고 자신이 시작한 서버만 종료합니다. DB가 준비되지 않으면 실패합니다.

브라우저에서는 지역 연쇄 선택, 회원 조건·이력, 관리자 장소 등록, 여행 2일 12단계·3일 18단계, 각 단계 길찾기를 확인합니다. 가상 장소는 지역 길찾기로 표시되어야 합니다.

**이번 README 작성에서는 저장소 소스와 문서를 확인했습니다. 실제 MySQL 실행·OpenAI 호출·브라우저 통합 테스트는 수행하지 않았습니다.** 기존 버전 문서의 과거 점검 기록과 이번 검증 범위를 구분합니다.

## 앞으로의 변경 관리

기존 화면과 기능을 유지하며 변경된 코드·스키마 변경 파일·문서만 커밋합니다. `.env`, API 키, DB 비밀번호, 사용자 데이터, 백업 파일은 올리지 않습니다. 각 변경에는 이유·적용 방법·DB 영향·검증 결과를 기록합니다.

---

## 기존 설치·백업·이관 상세 안내

아래는 기존 운영 안내입니다. 신규 설치·기존 DB 이관 시 참고하고, 이미 정상 실행 중인 V8.4 계열의 단순 화면 업데이트에는 DB 초기화 절차를 반복하지 않습니다.

### V8.4 보존형 전환 패키지 안내

이 패키지는 **V8.2 계열 사용자/관리자 화면**을 유지하면서, 독립 검증을 통과한 **V8.3 정규화 MySQL DB**로 API를 연결한 것입니다. **이전에 사용하던 `lunch_decision` DB에 UPDATE/ALTER/DELETE/DROP를 보내지 않습니다.**

## 구성

- 사용자 화면 `http://localhost:3100/`: 국가 → 시도 → 시군구 → 읍면동 → 상권, 회원가입/로그인, 개인 조건, 점심 추천/피드백/이력, 여행코스
- 관리자 화면 `http://localhost:3100/admin`: 행정구역 선택/등록, 음식점·카페·관광지·숙박 등록, 목록, FK 진단
- 백업: 원본/새 DB를 각각 읽기 전용으로 `.ndjson.gz`에 저장하고 SHA-256 체크섬 기록
- 이관: 원본을 읽기 전용으로 조회하고 **새 DB의 한 트랜잭션**으로 옮김. 이관 ID 대응표 `migration_mappings`로 재실행 중복 방지
- 중간 경로가 없는 상권 또는 원본 FK가 불명확하면 **임의로 다른 동에 붙이지 않음**: JSON 오류 보고서에 보류 기록
- DB 검증: MySQL FK, UI 선택 옵션 부모 ID, 전국/일본 샘플 지역까지 검사
- 테스트: JavaScript 계약 검사, 서버 시작 후 실제 HTTP API 통신 검사

## 설치 전 주의

1. **기존 V8.2.8과 V8.3 서버를 종료**하세요 (PowerShell `Ctrl+C`).
2. V8.4를 **새 폴더**에 압축 해제하세요. 기존 폴더를 덮어쓰지 마세요.
3. 기존 DB를 삭제하거나 기존 `database.sql`을 실행하지 마세요.
4. 새 DB 이름 `lunch_decision_v83`, 이전 DB 이름 `lunch_decision`을 **서로 다르게 설정**하세요.
5. API 키와 `.env`·백업 파일에는 비밀번호/개인정보가 포함될 수 있으므로 공유하지 마세요.

## Windows 11 PowerShell — 실행 순서

현재 저장소에서는 `lunch-decision-v8.4.3-map-layout-fixed` 안쪽 실행 폴더에서 실행합니다.

```powershell
npm install
Copy-Item .env.example .env
notepad .env
```

이미 `.env`를 만들었다면 `Copy-Item` 단계는 건너뛰세요. `.env` 예:

```env
PORT=3100
DB_HOST=localhost
DB_PORT=3306
DB_USER=root
DB_PASSWORD=실제_MYSQL_암호
OLD_DB_NAME=lunch_decision
V83_DB_NAME=lunch_decision_v83
SESSION_SECRET=적어도32글자이상의_랜덤_비밀키
ADMIN_USERNAME=admin
ADMIN_PASSWORD=본인만_아는_새_관리자암호
OPENWEATHER_API_KEY=
GOOGLE_MAPS_API_KEY=
```

### 1단계. 백업 — **먼저 실행**

```powershell
npm run backup
```

출력: `backups/latest-backup-manifest.json`과 `.ndjson.gz`. 기존/새 DB 백업은 읽기 전용으로 실행하며 DB 변경 없음. 압축된 자료와 manifest는 **반드시 PC 안에 보관**하세요.

### 2단계. 새 DB 준비와 샘플 테스트

```powershell
npm run setup
npm run seed:demo
npm run verify
```

`setup`은 오직 `lunch_decision_v83`을 준비합니다. 새로운 컬럼은 별도의 설치 명령에서만 추가합니다. 로그인이나 페이지 조회 시 ALTER가 발생하지 않습니다.

샘플 경로: 한국 서울특별시 종로구 → 종로1·2·3·4가동 → 종각상권 등 6개. 관광/점심/카페/관광/저녁/숙박 **시연용 가상 장소**가 포함됩니다. 테스트 장소의 주소·가격·좌표·영업시간은 실제 영업 정보가 아닙니다.

### 3단계. **이관 미리보기 — DB 변경 없음**

```powershell
npm run migrate:plan
```

결과 `diagnostics/migration-plan-report.json`을 확인하세요.

`skipped`(보류) 항목이 있다면 **상권이 기존 DB에서 시·군·구 바로 아래에 연결되는 등 정확한 하위 읍·면·동을 찾을 수 없는 경우**입니다. 다른 읍·면·동에 임의로 붙이지 않습니다.

### 4단계. **실제 이관**

```powershell
npm run migrate:apply
```

`migration_mappings`에 원본 ID와 신규 ID를 저장합니다. 실제 이관에 오류가 나면 **트랜잭션을 롤백**하고 기존 DB는 그대로 유지합니다.

이관 보류가 1건이라도 있으면 **기본 설정은 실제 커밋을 거절**합니다. 보류 내역을 확인하고 승인한 경우에만 다음을 사용하세요.

```powershell
node scripts/migrate.js --apply --allow-skips
```

이 옵션은 매핑되지 않은 항목을 새 DB에 포함하지 않습니다. 원본 데이터는 삭제되지 않습니다. 새 DB에 이관되지 않은 행은 `diagnostics/migration-apply-report.json`에 이유와 함께 저장됩니다.

**같은 아이디인데 비밀번호 해시가 다르거나 Google Place ID가 충돌하면 임의로 병합하지 않고 이관을 중단**합니다.

### 5단계. FK 연결 검사

```powershell
npm run verify
```

`diagnostics/connection-report.json`에서 `ok: true`, `foreignKeyErrors`의 값이 모두 `0`, `selectorBroken`가 `[]`인지 확인합니다.

### 6단계. 서버 및 화면 테스트

PowerShell 창 1:

```powershell
npm start
```

브라우저 주소: `http://localhost:3100`, 관리자: `http://localhost:3100/admin`

PowerShell 창 2에서 같은 폴더로 이동한 후:

```powershell
npm run test:ui
npm run test:api
```

`test:api`는 **읽기와 게스트 추천만 수행**하고 회원가입/저장은 자동으로 실행하지 않습니다. `diagnostics/screen-api-report.json`에 결과를 저장합니다.

### 화면 수동 테스트 (필수)

1. `http://localhost:3100` → 대한민국 → 서울특별시 → 종로구 → 종로1·2·3·4가동 → 종각상권 확인
2. 오늘 점심 다시 결정하기 → 등록된 테스트 음식점이 표시되는지 확인
3. 회원가입 → `회원등록 OK` 표시 → 조건 저장 → 나의 기록 다시 조회
4. `/admin` → `.env`의 관리자 계정으로 로그인 (기본 비밀번호를 강제로 사용하지 않음)
5. 음식점·여행장소 등록 → 상권을 선택한 후 등록 OK → 조회 목록에 보이는지 확인
6. `지역 관리 > 하위 지역 연결 검사`에서 잘못된 FK 수가 0인지 확인
7. 여행코스 → 종로구 선택 → 등록된 **테스트용** 관광/식사/카페/숙박이 6단계로 나타나는지 확인

## 이전 버전으로 돌아가기

- 새 V8.4 서버를 `Ctrl+C`로 종료하고 **원래 V8.2.8 폴더에서 이전 서버를 실행**하면 됩니다.
- `lunch_decision` 데이터는 이관 중 쓰지 않기 때문에 변경되지 않습니다. `lunch_decision_v83`은 별도 DB이며, V8.4에서 새로 저장한 데이터는 자동으로 옛 버전에 반영되지 않습니다.
- 백업 데이터를 따로 **새로운 빈 DB로 복구**하려면 다음 명령을 사용합니다. 기존 이름을 지정하면 실행하지 않습니다.

```powershell
node scripts/restore-to-new-db.js lunch_restore_20261016 lunch_decision
```

이 복원 명령은 **데이터와 일반 테이블을 복구**하며 VIEW·트리거·프로시저는 대상 원본의 구성에 따라 별도로 확인해야 합니다. 원본 DB 덮어쓰기는 지원하지 않습니다.

## 알려진 범위와 제한

- **실제 MySQL/브라우저 통합 테스트는 사용자의 PC에서 실행해야** 합니다. 제공된 ZIP의 Node 문법/순수 테스트만 제공 환경에서 확인했습니다.
- V8.4의 Google Places 자동 수집은 지역이 잘못 매핑되는 것을 방지하기 위해 **일시 비활성화(HTTP 501)** 되어 있습니다. 기존 데이터 및 관리자 수동 장소 입력은 가능합니다.
- 이동시간은 OSRM API에 실제 좌표가 있고 경로 조회가 성공했을 때만 표시합니다. 좌표가 없는 시연 장소는 `미확인`으로 표시합니다.
- 지역 구조가 정상이어도 해당 읍·면·동에 **상권 데이터가 없으면 상권 선택지는 비어 있습니다**. 관리자에서 상권을 추가하세요.
- 관리자·회원 비밀번호를 테스트 계정 그대로 운영 환경에 배포하지 마세요. HTTPS·세션 저장소·CSRF 대응은 운영 배포 전에 별도 구성해야 합니다.

### 진단 파일

- `backups/latest-backup-manifest.json` — 백업 데이터/해시
- `diagnostics/migration-plan-report.json` — 이관 미리보기
- `diagnostics/migration-apply-report.json` — 실제 이관 결과 및 보류 항목
- `diagnostics/connection-report.json` — DB/FK 선택 관계
- `diagnostics/screen-api-report.json` — 사용자/관리자 HTTP 연결 점검

**절대 `sql/01_schema.sql` 파일을 기존 DB로 직접 실행하지 마세요.** DB 이름이 내장되어 있지만, SQL은 `npm run setup` 경로로만 실행하기를 권장합니다.


## V8.4.1 업데이트 — `fetch failed` 진단 및 자동 시작

이전 `npm run test:api`는 별도 터미널에서 `npm start`를 켜야 했고 `PORT`를 무시한 채 `localhost:3100`으로 요청했습니다.
Windows에서 localhost가 IPv6(::1)으로 선택되면 IPv4(127.0.0.1) 서버로 연결하지 못하는 경우도 있었습니다.

V8.4.1은 다음을 변경했습니다.

- `.env`의 `PORT`를 API 테스트가 직접 사용하고, 기본 목적지로 `http://127.0.0.1:PORT`를 사용합니다.
- 서버가 꺼져 있으면 테스트용 서버를 자동 시작하고 테스트 후 **자신이 띄운 서버만 종료**합니다. 이미 실행 중인 서버는 종료하지 않습니다.
- 같은 포트에 다른 버전의 서버가 실행 중이면 버전을 확인하여 분명한 오류를 출력합니다.
- MySQL 접속·스키마 오류는 `fetch failed` 대신 URL/HTTP 상태/서버 오류를 표시합니다.
- `npm run doctor`를 추가했습니다. DB 비밀번호는 표시하지 않고 실행 포트와 DB 상태를 확인합니다.

압축을 새 폴더에 풀고 기존 `.env`를 복사한 뒤 다음만 실행해도 됩니다.

```powershell
npm install
npm run doctor
npm run test:ui
npm run test:api
```

서버를 계속 실행한 채 웹 화면을 사용하려면 별도로 `npm start`를 실행하세요.

주의: 서버를 자동으로 시작하더라도 DB가 준비되지 않았다면 테스트는 실패합니다. 이 경우 `npm run setup`, `npm run seed:demo`, `npm run verify` 결과를 먼저 확인하세요.

`API_TEST_AUTOSTART=0`으로 설정하면 자동 서버 시작을 끌 수 있습니다. `TEST_BASE_URL`로 원격/별도 포트를 선택할 수도 있습니다.

