# v23-render-mobile-install 변경사항

- 앱 설치 안내 문구를 PC 기준이 아니라 모바일 기준으로 정리했습니다.
- Android는 Chrome 설치창 또는 ⋮ 메뉴 > 앱 설치/홈 화면에 추가 방식으로 안내합니다.
- iPhone/iPad는 Safari 공유 버튼 > 홈 화면에 추가 방식으로 안내합니다.
- Render 무료 Web Service 배포용 `render.yaml`, `.env.render.example`, `RENDER_FREE_DEPLOY.md`를 추가했습니다.
- API 버전은 `v23-render-mobile-install`입니다.

# 제약뉴스 RSS 대시보드 React/Node 로컬 버전

## v22-install-simple-onlineprep 변경사항

- PC Dashboard의 카테고리 분포 카드에서 아래 공백을 줄이고 막대 그래프 두께/간격을 확대했습니다.
- 모바일 접힘 구조는 유지하고, PC 화면에서만 카테고리 분포 시각화를 크게 보이도록 조정했습니다.

## v19-pwa-install-mobile-fold 변경사항

- 실행 안정화 기준: API `127.0.0.1:8790`, React `127.0.0.1:5190`
- 키워드 인텔리전스/규제기관 정책 탭 제거
- 규제기관 공식자료 대시보드는 외부 연동 버튼만 유지
- 회수·처분 모니터링 탭은 회수·판매중지, 행정처분·영업정지, 허가취소·품목정지만 표시
- GMP·품질위험 카드는 회수·처분 모니터링 탭에서 제거
- RSS 2차 수집 보정: RSS 검색식 때문에 일반 기사가 회수/처분으로 분류되는 문제 완화
- `데일리팜 뉴스` 같은 잡음성 RSS 항목 제외
- 유사 이슈 묶음 대표 제목 클릭 시 원문 링크 열기
- 화면 글자 크기/카드 밀도 축소

자세한 구현 차이는 `docs/IMPLEMENTATION_STAGE.md`를 확인하십시오.

---

# 제약뉴스 RSS 대시보드 React PWA 전환 MVP

## v8 schemafix 변경사항

- 기존 Supabase `news_articles` 테이블에 `article_summary`, `article_text`, `sub_tags`, `classification_reason`, `classification_score`, `body_fetch_status` 컬럼이 없어도 조회가 중단되지 않도록 API 서버에 컬럼 호환 fallback을 추가했습니다.
- 서버가 처음에는 전체 컬럼을 조회하고, Supabase가 `column ... does not exist` 오류를 반환하면 해당 컬럼을 제외한 뒤 자동 재조회합니다.
- 검색 조건도 실제 사용 가능한 컬럼에 대해서만 적용합니다. 따라서 기존 Streamlit 캐시 테이블을 바로 연결해도 React 화면이 표시됩니다.
- 장기적으로는 `supabase_schema.sql`의 `alter table ... add column if not exists ...` 구문을 Supabase SQL Editor에서 1회 실행하면 신규 컬럼까지 사용할 수 있습니다.


현재 사용 중인 Streamlit `v34.zip`를 기준 기능 명세로 보고, 다음 구현 단계인 **React + Node.js 조회형 PWA MVP**로 전환한 버전입니다.

## 이 버전의 목적

- Streamlit rerun 구조에서 벗어나 탭 전환/페이지 이동 속도 확인
- Supabase 최근 30일 캐시 데이터를 React 화면에서 빠르게 조회
- 모바일 PWA 적용 가능성 확인
- Node.js API 서버를 통해 Supabase `secret/service_role key`를 브라우저에 노출하지 않음

## 포함 범위

### 포함
- React + Vite 프론트엔드
- Node.js + Express API 서버
- Supabase `news_articles`, `collection_log` 조회
- Dashboard
- 뉴스목록 타임라인 + 페이지네이션
- 카테고리 인텔리전스
- 규제 레이더
- 규제기관 정책 탭
- 모바일 반응형 UI
- PWA manifest / service worker 기본 구성

### 아직 제외
- Node.js RSS 수집 API 실제 구현
- Google News RSS 병렬 수집
- 기사 본문 보강 수집
- 자동 수집 스케줄

현재 RSS 수집/분류/Supabase 저장은 기존 Streamlit v34를 계속 사용하고, 이 React PWA는 **조회 전용**으로 먼저 속도와 UI를 검증합니다.

## 폴더 구조

```text
pharma-news-pwa-v7-local-clientfix/
├─ server/              # Node API 서버
├─ client/              # React PWA 화면
├─ supabase_schema.sql  # 기존 Supabase 테이블 스키마
├─ .env.example
└─ docs/
```


## 이번 로컬 수정본의 핵심

- `localhost` 대신 `127.0.0.1` 기준으로 API/React/프록시를 통일했습니다.
- 이전 버전 Service Worker 또는 브라우저 캐시가 빈 화면을 만드는 문제를 피하기 위해 `reset.html`을 추가했습니다.
- `run_local.bat`은 API와 React가 실제 응답할 때까지 각각 대기하고, 실패 시 `server.log` 또는 `client.log`를 출력합니다.
- React 개발 모드에서는 Service Worker를 등록하지 않고 기존 PWA 캐시를 제거합니다.

## 사용 방법: 로컬 실행

### 1. 압축 해제

```bash
unzip pharma-news-pwa-v7-local-clientfix.zip
cd pharma-news-pwa-v7-local-clientfix
```

### 2. 환경변수 생성

`.env.example`을 복사해서 `.env`를 만듭니다.

```bash
cp .env.example .env
```

`.env` 내용을 실제 Supabase 값으로 수정합니다.

```env
PORT=8790
CORS_ORIGIN=http://127.0.0.1:5190,http://localhost:5190
SUPABASE_URL=https://xxxxx.supabase.co
SUPABASE_SERVICE_KEY=sb_secret_xxxxx
CACHE_DAYS=30
INITIAL_DAYS=7
```

`SUPABASE_SERVICE_KEY`에는 Supabase `service_role key` 또는 새 체계의 `secret key`를 넣습니다. GitHub에 올리면 안 됩니다.

### 3. 패키지 설치

```bash
npm run install:all
```

### 4. 개발 서버 실행

```bash
npm run dev
```

실행 주소:

```text
React 화면: http://127.0.0.1:5190
Node API:   http://127.0.0.1:8790/api/health
```

## Streamlit v34와 같이 쓰는 방식

1. 기존 Streamlit v34에서 RSS 수집 버튼 실행
2. Supabase `news_articles`에 최근 30일 캐시 저장
3. React PWA가 Supabase 캐시를 Node API를 통해 조회
4. React 화면에서 탭 전환/필터/페이지 이동 속도 확인

## 배포 방향

### 권장 구조

```text
Frontend: Vercel / Netlify / GitHub Pages + API proxy 제외 시 별도 설정 필요
Backend: Render / Railway / Fly.io / Supabase Edge Function 검토
DB: Supabase
```

초기 검증은 로컬 또는 사내 PC에서 먼저 하는 것을 권장합니다.

## 다음 단계

1. React 조회형 MVP 속도 확인
2. Node.js `POST /api/collect` 실제 수집 구현
3. Google News RSS 병렬 수집
4. Supabase 자동 upsert 및 30일 초과 삭제
5. PWA 아이콘/오프라인 UX 고도화
6. 자동 수집 스케줄 적용


## Windows BAT 실행

- `run_local.bat` : 필요한 패키지를 확인/설치한 뒤 Node API와 React 화면을 실행합니다.
- `stop_local.bat` : 로컬 개발 서버에서 사용하는 5190/8787/8790 포트의 node 프로세스를 종료합니다. 8787은 이전 로컬 빌드 잔여 프로세스 정리용입니다.

기존 의약품 챗봇 등 다른 Vite 앱과 충돌하지 않도록 이 프로젝트는 다음 포트를 사용합니다.

```text
React 화면: http://127.0.0.1:5190
Node API:   http://127.0.0.1:8790/api/health
```


## Local port fix note

주의: 이전 버전의 .env에 PORT=8787이 남아 있어도 start_server.bat에서 PORT=8790을 강제 적용합니다.


## v19 메모
- 전용 홈페이지 크롤러는 도입하지 않았습니다. Google News RSS + site: 검색식 기반 수집만 유지합니다.
- 제조업무정지, 영업정지, 행정처분, 회수, 판매중지, 품목정지, 허가취소 등 실제 조치성 근거가 있으면 중요도를 '높음'으로 화면 보정합니다.
- 과거 Supabase 캐시에 '회수/처분 + 중간'으로 남은 행도 조회 시 '높음'으로 정합성 보정합니다.


## v19 PWA/mobile stage 1

- 모바일 하단 탭바와 카드형 레이아웃을 1차 반영했습니다.
- manifest, PNG 아이콘, maskable icon, apple-touch-icon, service worker를 정리했습니다.
- 개발 모드(run_local.bat)는 캐시 꼬임 방지를 위해 service worker를 해제합니다.
- PWA 설치성 확인은 run_pwa_preview.bat로 React build 후 http://127.0.0.1:8790에서 확인합니다.
- API 연결 실패 시 마지막 성공 조회 스냅샷을 표시합니다.


## v22 앱 설치 단순화 및 온라인화 준비 단계
- 앱 설치 버튼 클릭 시 설치 프롬프트가 없으면 진단 패널을 표시합니다.
- Manifest, Service Worker, 보안 컨텍스트, 설치 프롬프트 준비 여부를 화면에서 확인할 수 있습니다.
- 캐시 초기화 버튼으로 이전 PWA 캐시/스냅샷을 정리하고 새로고침할 수 있습니다.
