# 변경 이력 (Changelog)

이 프로젝트의 모든 주요 변경 사항은 이 파일에 문서화됩니다.

이 형식은 [Keep a Changelog](https://keepachangelog.com/en/1.0.0/)를 기반으로 하며,
이 프로젝트는 [Semantic Versioning](https://semver.org/spec/v2.0.0.html)을 준수합니다.


## [5.8.8] - 2026-06-02

### 🐛 버그 수정 (Bug Fix)

- **검색 필터 버그 수정 (search-filter-fix)**: 단일 검색어에서 누락되던 필드 및 IES 서비스 검색 불가 문제 4건 수정
  - Epipe/VPLS SAP `description`, `portDescription` 단일 검색어에서 누락 수정
  - Epipe/VPLS Spoke/Mesh SDP `description` 단일 검색어에서 누락 수정
  - IES 서비스를 hostname·serviceId·description으로 검색 불가 버그 수정 (서비스 레벨 매칭 분리)
  - 단일 검색어 Catch-all에서 Unicode 하이픈 정규화(`normalizeSearchString`) 미적용 수정
  - VPRN 인터페이스 `portDescription` 명시적 검색 추가 (단일/복수 경로 모두)

## [5.8.1] - 2026-04-10

### ⚡ 성능 (Performance)

- **React 성능 최적화 v2 (Vercel React Best Practices 재점검)**: v5.8.0 이후 잔존 이슈 4종 해결
  - `groupedDiagrams` useMemo 추출 → JSX 내 inline `diagrams.reduce()` 제거, 매 렌더마다 O(n) 재계산 방지
  - `handleCopyCode` useCallback 적용 (`[diagram]` 의존성) → `handleCopyImagePNG`와 일관성 확보
  - `handleZoomIn/handleZoomOut/handleResetZoom` useCallback 적용 (`[]` 의존성) → functional updater로 안전한 빈 의존성
  - `Dashboard` `STAT_CARD_DEFS` 컴포넌트 외부 상수화 → 정적 설정과 동적 데이터 분리, `statCards` useMemo 불필요 제거

## [5.8.0] - 2026-04-10

### ⚡ 성능 (Performance)

- **React 성능 최적화 (Vercel React Best Practices)**: 코드베이스 전체 점검 후 5종 이슈 일괄 해결
  - `resize`/`stopResizing`/`startResizing` useCallback 적용 → 이벤트 리스너 메모리 누수 수정
  - WebSocket `handleMessageRef` 패턴 적용 → Stale Closure 버그 수정
  - `src/utils/stringUtils.ts` 신규 생성 → `normalizeSearchString` 공유 유틸 추출, 정규식 모듈 레벨 호이스팅
  - `handleCopyImagePNG` useCallback 적용
  - `statCards` useMemo 적용

## [5.7.3] - 2026-04-10

### ✨ 개선 (Enhancement)

- **Show Code → Copy Code 버튼 변환**: 기존 2단계(코드 표시 → Copy 클릭) → 1단계(Copy Code 클릭 즉시 복사), 복사 후 2초간 초록색 `Copied!` 피드백 표시

## [5.7.1] - 2026-04-03

### 🐛 버그 수정 (Bug Fixes)

- **포털 iframe Nokia Visualizer 접속 불가 수정**:
  - `Sec-Fetch-Dest: iframe` 헤더로 iframe 컨텍스트 감지
  - 인증 실패 시 iframe 내부 리다이렉트 대신 TOP 프레임을 Nokia Visualizer로 이동 (`?_pr=1`)
  - `auth.hub.sk-net.com` X-Frame-Options 차단 우회
  - [nginx.conf](nginx.conf)
- **`?_pr=1` 포털 자동 복귀 처리**:
  - Nokia Visualizer 인증 완료 후 `?_pr=1` 감지 시 포털 `/visualizer`로 즉시 복귀
  - React 렌더링 이전 동기 처리로 화면 깜빡임 없음
  - [src/main.tsx](src/main.tsx)

## [5.7.0] - 2026-04-02

### 🔒 보안 (Security)

- **Authentik SSO Forward Auth 인증 게이트 구현**:
  - nginx `auth_request`로 Authentik Embedded Outpost 연동, 미인증 접근 전면 차단
  - 모든 경로 적용 (`/`, `/api/*`, `/ws`, 정적 파일)
  - `/health` 엔드포인트 인증 예외 (포털 모니터링용)
  - [nginx.conf](nginx.conf)
- **포털 iframe 임베딩 허용**:
  - `X-Frame-Options: SAMEORIGIN` 제거
  - CSP `frame-ancestors 'self' https://portal.hub.sk-net.com` 추가
- **SSO 쿠키 도메인 공유 (`hub.sk-net.com`)**:
  - Authentik Proxy Provider `cookie_domain=hub.sk-net.com` 설정
  - 포털 로그인 1회로 Nokia Visualizer iframe 자동 인증
- **포털 헬스체크 URL 수정**:
  - Nokia Visualizer 헬스체크를 `/health` 엔드포인트로 변경 (인증 우회)

## [5.6.3] - 2026-03-31

### 🐛 버그 수정 (Bug Fixes)

- **Mermaid 노드 라벨 HTML 속성 따옴표 충돌 수정**:
  - 노드 라벨 `["..."]` 내부 HTML 속성의 이중 따옴표 → 단일 따옴표로 변경 (Grafana 렌더링 깨짐 해결)
  - `qosHighlight()` span 태그 및 전체 div 라벨 적용 (Epipe/VPLS/VPRN/BGP/OSPF/STATIC)
  - [src/utils/v3/mermaidGeneratorV3.ts](src/utils/v3/mermaidGeneratorV3.ts)
- **노드 ID 하이픈 파싱 오류 수정**:
  - `sanitizeNodeId()`에서 하이픈도 언더스코어로 변환 (`SK-Net` → `SK_Net`)
  - Mermaid가 서브그래프/노드 ID의 하이픈을 화살표 문법으로 오파싱하는 문제 해결
  - [src/utils/v3/mermaidGeneratorV3.ts](src/utils/v3/mermaidGeneratorV3.ts)


## [5.6.2] - 2026-03-21

### 🐛 버그 수정 (Bug Fixes)

- **Dashboard 검색 Unicode 하이픈 정규화 수정**:
  - Dashboard 검색창에서 U+2011 Non-Breaking Hyphen(`‑`) 포함 hostname 검색 불가 문제 해결
  - `siteName`, `hostnames` 필터에 `normalizeSearchString()` 적용 (ServiceListV3와 동일 로직)
  - [src/components/v3/Dashboard.tsx](src/components/v3/Dashboard.tsx)


## [5.6.1] - 2026-03-19

### 🐛 버그 수정 (Bug Fixes)

- **networkType 덮어쓰기 버그 수정**:
  - 프론트엔드 `useConfigSync`가 `POST /api/ncv/analyze` 호출 시 configStore의 networkType 데이터를 덮어쓰는 문제 해결
  - `POST /analyze` 엔드포인트에서 기존 networkType을 보존하고 incoming 서비스에 적용
  - [server/src/routes/ncv.ts](server/src/routes/ncv.ts) - preservedNetworkType 로직

- **ISP/MPLS 다이어그램 분리**:
  - 동일 Service ID (VPLS 4073 등)가 다른 망에 있을 때 All 필터에서도 별도 다이어그램으로 표시
  - `serviceGroups` 그룹핑 키에 networkType suffix 추가
  - 다이어그램 렌더링 그룹핑에도 networkType 반영
  - [src/pages/V3Page.tsx](src/pages/V3Page.tsx) - serviceGroups, diagram rendering 수정

- **프론트엔드 networkType 전파**:
  - `GET /api/config/file/:filename` 응답에 `X-Network-Type` 헤더 추가
  - 파일 로드 시 networkType을 파서에 전달하여 서비스별 networkType 설정
  - [server/src/routes/config.ts](server/src/routes/config.ts), [src/pages/V3Page.tsx](src/pages/V3Page.tsx)

### 🚀 UI 개선 (Enhancements)

- **다이어그램 networkType 배지**:
  - ServiceDiagram 헤더에 ISP (Cyan) / MPLS (Purple) / Cloud (Slate) 배지 표시
  - Service Group 헤더에도 망 구분 배지 추가
  - [src/components/v3/ServiceDiagram.tsx](src/components/v3/ServiceDiagram.tsx)

- **망 구분 필터 버튼 제거**:
  - ServiceListV3 좌측 패널에서 ISP/MPLS/CLOUD/UNKNOWN 필터 버튼 삭제
  - 다이어그램 자체에 망 구분이 표시되므로 별도 필터 불필요
  - [src/components/v3/ServiceListV3.tsx](src/components/v3/ServiceListV3.tsx)

### 📝 문서화

- **PDCA Plan**: `docs/01-plan/features/vpls4073-networktype-overwrite.plan.md`

---

## [5.6.0] - 2026-03-19

### 🚀 신규 기능 (Features)

#### Network Type Separation (Phase 1)
- **Network Type 분류**: ISP/MPLS/Cloud 망 자동 구분
  - 파일 경로 기반 networkType 자동 추출 (`/configs/isp/`, `/configs/mpls/`, `/configs/cloud/`)
  - Hybrid 전략: 경로 → 파일명 → Config 내용 추론
  - [networkTypeExtractor.ts](server/src/utils/networkTypeExtractor.ts) - extractNetworkType(), determineNetworkType()

- **selectionKey 생성 로직 개선**:
  - 이전: `${serviceType}-${serviceId}` (예: `vpls-4073`)
  - 신규: `${serviceType}-${serviceId}-${networkType}` (예: `vpls-4073-isp`, `vpls-4073-mpls`)
  - Fallback: networkType이 `unknown`이면 기존 포맷 사용 (하위 호환)

- **Service ID 충돌 해결**:
  - 사례: VPLS 4073 ISP (`SVC_SKSH_Internet_2G`) vs VPLS 4073 MPLS (`Gyowon_Access-Control`)
  - 결과: 독립된 토폴로지 뷰 생성 (ISP 망, MPLS 망 분리)

- **Frontend UI 추가**:
  - [ServiceListV3.tsx](src/components/v3/ServiceListV3.tsx) - Network Type 필터 버튼 (ISP/MPLS/CLOUD)
  - Network Type 배지 (Cyan=ISP, Purple=MPLS, Slate=Cloud)
  - HA 감지 로직 networkType 반영 (동일 serviceId + networkType HA 그룹)

### 🔧 Backend 구조 개선

- **타입 정의 추가**:
  - [src/types/services.ts](src/types/services.ts) - NetworkType 타입 (`'isp' | 'mpls' | 'cloud' | 'unknown'`)
  - [server/src/types.ts](server/src/types.ts) - ServiceSummary.networkType 필드
  - [server/src/services/configStore.ts](server/src/services/configStore.ts) - StoredConfig.networkType

- **Parser 통합**:
  - [src/utils/v3/parserV3.ts](src/utils/v3/parserV3.ts) - ParseOptions 인터페이스 추가
  - [server/src/services/nokiaParserCore.ts](server/src/services/nokiaParserCore.ts) - ParseOptions 파라미터 지원
  - [server/src/services/autoParser.ts](server/src/services/autoParser.ts) - extractNetworkType 적용

- **Config Summary Builder**:
  - [src/utils/configSummaryBuilder.ts](src/utils/configSummaryBuilder.ts) - generateSelectionKey 함수 추가
  - 모든 서비스에 networkType 필드 반영

### 📝 문서화

- **PDCA 문서 생성**:
  - [docs/01-plan/features/network-type-separation.plan.md](docs/01-plan/features/network-type-separation.plan.md)
  - [docs/02-design/features/network-type-separation.design.md](docs/02-design/features/network-type-separation.design.md)

### 🧪 검증 (Verification)

- **파싱 결과**: 94개 Nokia config, 782개 서비스
- **VPLS 4073 분리 확인**:
  - `vpls-4073-mpls` → Gyowon_Access-Control
  - `vpls-4073-isp` → SVC_SKSH_Internet_2G_1/2
- **TypeScript 컴파일**: 오류 없이 성공 ✅
- **Docker 배포**: 재빌드 및 재시작 완료 ✅

### 📦 배포 정보

- **Branch**: `feature/network-type-separation-phase1` → `main` (merge 필요)
- **Docker Images**:
  - `nokia-visualizer-nokia-visualizer:latest`
  - `nokia-visualizer-nokia-api:latest`
- **Git Commits**:
  - `3e49b52` - Backend implementation
  - `f33a52c` - Frontend implementation
  - `4a8e6f6`, `2d87880`, `9bde22f` - TypeScript fixes

### 🔄 마이그레이션

- **하위 호환성**: 기존 서비스는 networkType `unknown` 또는 생략 가능
- **자동 마이그레이션**: 파일 경로 재파싱 시 networkType 자동 할당
- **Fallback 키 생성**: networkType이 없거나 `unknown`이면 기존 selectionKey 포맷 사용

### 🎯 향후 계획

- **Phase 2 (v5.7.0)**: Parser 모듈화 리팩토링 (1678줄 → 14개 모듈)
- **Phase 3 (v5.8.0)**: Network Type 기반 통계 대시보드


## [5.5.2] - 2026-03-19

### 🐛 버그 수정 (Bug Fixes)

#### Unicode 하이픈 문자 검색 오류 수정
- **문제**: `SK‑Net_Gyowon` (U+2011 Non-Breaking Hyphen) 같은 특수 하이픈 포함 hostname 검색 불가
- **원인**: JavaScript `.toLowerCase().includes()` 는 Unicode 하이픈 정규화 미지원
- **해결**:
  - [ServiceListV3.tsx:55-65](src/components/v3/ServiceListV3.tsx#L55-L65) - `normalizeSearchString()` 헬퍼 함수 추가
  - Unicode 하이픈 변형 7종 (`U+2010~U+2015`, `U+2212`, `U+FE58`, `U+FE63`, `U+FF0D`) → ASCII hyphen (`U+002D`) 통일
  - NFKD 정규화로 호환 문자 자동 처리
  - 단일/복수 검색어, IES 인터페이스 필터링 모두 적용

### 🧪 테스트 (Testing)

- **Unicode 하이픈 검증 테스트**: 6개 테스트 케이스 100% 통과
  - `SK‑Net_Gyowon_7705SAR‑8_MPLS` hostname 검색 정상화
  - 일반 하이픈 hostname 호환성 유지

### 📝 영향 범위 (Impact)

- **검색 정확도 개선**: 특수 문자 포함 hostname 검색 정상화
- **호환성**: 기존 일반 하이픈 검색 100% 호환
- **성능**: 정규화 오버헤드 < 1ms (무시 가능)


## [5.5.1] - 2026-03-16

### 🐛 버그 수정 (Bug Fixes)

#### Copy PNG 클립보드 복사 기능 수정
- **문제**: 'Copy PNG' 버튼이 클립보드 복사 대신 다운로드 기능으로 작동
- **원인**: NPM(nginx proxy manager)의 Permissions Policy로 Clipboard API 차단
- **해결**:
  - Clipboard API 우선 시도, 실패 시 다운로드 fallback 추가
  - [ServiceDiagram.tsx:218-248](src/components/v3/ServiceDiagram.tsx#L218-L248) 개선된 에러 처리
  - [nginx.conf](nginx.conf#L44) - Docker 컨테이너에 `Permissions-Policy` 헤더 추가
  - NPM 프록시 설정 가이드 추가: `add_header Permissions-Policy "clipboard-write=(self), clipboard-read=(self)" always;`

### 📚 문서 개선 (Documentation)

- **릴리즈 노트 디렉토리 구조화**: `docs/releases/` 신규 생성
- **NPM 프록시 Clipboard API 설정 가이드** 추가

### 🔧 기술 개선 (Technical Improvements)

- **Clipboard API 에러 처리 강화**:
  - 상세한 콘솔 로그 출력 (Blob 생성, 복사 성공/실패)
  - Permissions Policy 차단 시 사용자 친화적 안내 메시지
  - Chrome, Safari 호환 Permissions-Policy 문법 적용


## [5.5.0] - 2026-03-13

### ✨ 신규 기능 (New Features)

#### AutoParser Service (자동 파싱)
- **FileWatcher 기반 자동 Config 파싱** (`server/src/services/autoParser.ts`):
  - 서버 시작 시 `/data/configs` 폴더의 모든 Nokia config 자동 파싱
  - FileWatcher 이벤트 (`file-added`, `file-changed`, `file-deleted`) 기반 실시간 ConfigStore 업데이트
  - ConfigStore 항상 최신 상태 유지 (94개 config, 782개 서비스 자동 로드)
- **Nokia Parser 백엔드 포팅** (`server/src/services/nokiaParserCore.ts`):
  - Frontend `parserV3.ts` (1,677줄) → Backend로 복사
  - Backend에서 직접 Nokia config 파싱 가능
  - Thin wrapper `nokiaParser.ts`로 간결한 API 제공

#### Vendor Detection 개선
- **Vendor Detector Service** (`server/src/services/vendorDetector.ts`):
  - Nokia TiMOS-B (7210 SAS) 인식 추가
  - netdevops-portal 방식 채택 (복잡한 regex → 단순 키워드 매칭)
  - Nokia/Arista/Cisco/Juniper 패턴 단순화로 유지보수성 향상

#### Telegram Bot 개선
- **Telegram Bot Service** (`server/src/services/telegramBot.ts`):
  - Markdown 파싱 오류 수정 (특수문자 자동 이스케이프)
  - ConfigStore 데이터 매핑 수정 (AutoParser 연동)
  - 응답 형식 간결화 (불필요한 구분선, 정확도 표시 제거)
  - 웹 UI와 일관된 사용자 경험 제공

#### AI 응답 품질 개선
- **시스템 프롬프트 강화** (`server/src/prompts/systemPrompt.ts`):
  - 리스트 형식 가이드 추가 (구조화된 데이터는 리스트로 표현)
  - 좋은 예시/나쁜 예시 제공
  - 포트/인터페이스/서비스 나열 시 가독성 대폭 향상

#### Web UI 개선
- **AIChatPanel 리스트 형식 지원** (`src/components/v3/AIChatPanel.tsx`):
  - `whitespace-pre-line` CSS 적용으로 AI 응답 줄바꿈 보존
  - Telegram 봇과 동일한 리스트 형식 표시

### 🔧 기술 개선 (Technical Improvements)

- **ConfigStore 확장** (`server/src/services/configStore.ts`):
  - `delete()` 메서드 추가 (파일 삭제 이벤트 처리)
- **Recursive Scanner** (`server/src/services/recursiveScanner.ts`):
  - 재귀 디렉토리 스캔 유틸리티 추가
- **index.ts 개선** (`server/src/index.ts`):
  - AutoParser 시작 로직 추가
  - FileWatcher 초기화 완료 후 자동 파싱 시작
- **Frontend 빌드 수정**:
  - Stale `.js/.d.ts` 파일 제거로 Vite 빌드 오류 해결
  - TypeScript 파일을 Vite가 직접 처리하도록 개선

### 📚 문서 업데이트

- **Plan 문서** (`docs/01-plan/features/auto-config-loading-v2.plan.md`)
- **Design 문서** (`docs/02-design/features/auto-config-loading-v2.design.md`)
- **Release Notes** (`RELEASE_NOTES_v5.4.0.md`)

### 🎯 주요 성과

- **Telegram 봇과 웹 UI 답변 일관성**: ConfigStore 자동 동기화로 동일한 데이터 사용
- **운영 편의성 향상**: Config 파일 추가/수정 시 자동 반영 (수동 업로드 불필요)
- **AI 응답 가독성 대폭 개선**: 리스트 형식 도입으로 구조화된 데이터 표현

### 🐛 버그 수정 (Bug Fixes)

- Telegram Markdown 파싱 오류 수정 (특수문자 이스케이프)
- ConfigStore 데이터 매핑 오류 수정
- Frontend 빌드 실패 오류 수정 (stale JS 파일 제거)


## [5.4.0] - 2026-03-11

### ✨ 신규 기능 (New Features)

- **v4 AI Platform 통합** (v5 베이스 유지):
  - v4-development 브랜치의 AI 기능을 v5로 이식
  - v5 UI/보안 개선사항 그대로 유지 (Tailwind CSS, Dashboard, 보안 정책)
  - 외부 시스템 연동을 위한 AI 협업 미들웨어 기능 추가

#### 백엔드 AI 기능
- **NCV REST API** (`server/src/routes/ncv.ts`):
  - 8개 엔드포인트: `/api/ncv/analyze`, `/services`, `/services/:key`, `/topology`, `/devices`, `/search`, `/export`, `/stats`
  - X-API-Key 인증 적용 (v5 보안 정책 준수)
- **ConfigStore** (`server/src/services/configStore.ts`):
  - In-Memory 파싱 결과 저장소 (싱글톤)
  - 서비스 검색, 통계, 토폴로지 생성 지원
- **MCP Server** (`server/src/mcp-server.ts`, `server/src/services/mcpTools.ts`):
  - Model Context Protocol 지원 (Claude Desktop, Cursor 등 AI 도구 연동)
  - HTTP MCP 엔드포인트 (`/mcp`)
  - 7개 MCP 도구: `get_services`, `get_service_detail`, `get_topology`, `search_config`, `get_devices`, `get_stats`, `get_ha_pairs`

#### 프론트엔드 통합
- **useConfigSync 훅** (`src/hooks/useConfigSync.ts`):
  - Config 로드 시 백엔드 ConfigStore 자동 동기화
  - V3Page에서 `useConfigSync(configs)` 1줄로 활성화
  - Demo 모드 또는 백엔드 미연결 시 조용히 무시

### 🔧 기술 개선 (Technical Improvements)

- **의존성 추가**:
  - `@types/html2canvas` v0.5.35 추가 (타입 안전성 개선)
- **Git 브랜치 통합**:
  - v4-development → main 선택적 머지 (AI 파일만)
  - v5 UI 아키텍처 완전 보존

### 📚 문서 업데이트

- **Plan 문서 작성** (`docs/01-plan/features/v4-ai-integration.plan.md`):
  - v4→v5 통합 전략 상세 문서화
  - Executive Summary, 목표, 범위, 위험 요소, 성공 기준 포함

### 🎯 통합 결과

- ✅ v5 Tailwind CSS UI 유지
- ✅ v5 보안 정책 유지 (CORS, API Key, Rate Limiting)
- ✅ v4 AI Platform 기능 완전 이식
- ✅ 프론트엔드/백엔드 빌드 성공
- ✅ 단일 main 브랜치로 통합 완료

---

## [5.3.1] - 2026-03-11

### ✨ 신규 기능 (New Features)

- **CORS origin 다중 허용 지원** (`server/src/index.ts`, `server/src/services/websocket.ts`):
  - 쉼표로 구분된 여러 origin 동시 허용 (예: `https://ncv.hub.sk-net.com,https://nokia-int.hub.sk-net.com`)
  - Express CORS 미들웨어 및 WebSocket `verifyClient`에서 배열 기반 검증 로직 적용
  - 다중 프로덕션 도메인 환경에서 안전한 CORS 제어
- **자동 업데이트 스크립트 추가** (`scripts/auto-update.sh`):
  - GitHub 최신 커밋 감지 시 자동 pull 및 Docker 재빌드
  - cron 작업 등에 활용 가능한 무인 배포 스크립트

### 🔧 기술 개선 (Technical Improvements)

- **docker-compose.yml 프로덕션 설정** (v5.3.0 후속):
  - `portainer-network` 연결로 portal-backend 헬스체크 통합
  - CORS_ORIGIN 환경변수 기본값을 프로덕션 도메인으로 변경
  - 볼륨 마운트 경로를 절대 경로(`/data/configs`)로 표준화

---

## [5.3.0] - 2026-03-09

### ✨ 신규 기능 (New Features)

- **폴더 선택 기능** (`ConfigFileList.tsx`, `FolderPathSettings.tsx`):
  - OS 파일 브라우저(`webkitdirectory`)로 로컬 폴더 직접 선택
  - 선택한 폴더의 `.txt` 파일 전체 자동 로딩 (서브디렉토리 포함)
  - 선택 폴더 정보(이름, 파일 수, 날짜)를 `localStorage`에 저장 — 재접속 시에도 유지
  - Config 파일 목록 헤더에 저장된 폴더 배지 표시

### 🐛 버그 수정 (Bug Fix)

- **폴더 선택 이중 모달 제거** (`ConfigFileList.tsx`):
  - "폴더 선택" 버튼 클릭 시 모달 없이 OS 파일 브라우저 바로 열림
  - `folderInputRef` 직접 트리거 방식으로 단계 제거
- **서브디렉토리 파일 인식** (`ConfigFileList.tsx`):
  - `webkitdirectory` 속성을 `useEffect` + `setAttribute`로 안정적으로 적용
  - `webkitRelativePath` 기준 정렬로 서브디렉토리 계층 구조 파일 처리

---

## [5.2.6] - 2026-03-03

### ✨ 신규 기능 (New Features)

- **대시보드 전체 서비스 수 표시** (`Dashboard.tsx`):
  - 요약 정보 줄에 전체 서비스 합계(Epipe + VPLS + VPRN + IES) 추가
  - Layers 아이콘과 함께 강조 표시
  - 기존 devices / HA pairs 항목과 동일한 스타일로 통일

---

## [5.2.5] - 2026-03-03

### 🐛 버그 수정 / 🎨 UI 개선 (Bug Fix / UI Improvement)

- **이중 스크롤바 제거** (`ServiceListV3.tsx`):
  - Epipe/VPLS/VPRN/IES 각 그룹 내부의 `max-h-[calc(100vh-300px)] overflow-y-auto` 제거
  - 외부 단일 스크롤 컨테이너(`flex-1 overflow-y-auto`)만 사용하는 글로벌 스탠다드 구조로 변경
  - VS Code, GitHub, Linear 등과 동일한 단일 스크롤 패턴 적용
- **그룹 헤더 Sticky 고정** (`ServiceListV3.tsx`):
  - 스크롤 중에도 어느 타입의 서비스인지 항상 표시 (`sticky top-0 z-10`)
  - Epipe(blue) / VPLS(emerald) / VPRN(violet) / IES(amber) 색상 보더와 함께 고정

---

## [5.2.4] - 2026-03-03

### 🎨 UI/UX 개선 (UI/UX Improvements)

- **서비스 타입별 Visual Identity 통합** (`ServiceListV3.tsx`):
  - 타입 필터 버튼 색상을 Dashboard 통계 카드와 동일 체계로 통일
    - All: gray / Epipe: blue / VPLS: emerald / VPRN: violet / IES: amber / 이중화: green
    - 활성 버튼: 진한 배경 + 흰 텍스트 / 비활성 버튼: 옅은 배경 + 타입 색상 텍스트
  - 그룹 헤더 좌측에 타입 색상 `border-l-4` 보더 추가 (Epipe: blue, VPLS: emerald, VPRN: violet, IES: amber)
  - 아이콘 중복 해소: VPLS `🌐` → `🔀` (L2 multipoint 스위칭), IES `🌐` → `🌍` (인터넷 서비스)
  - 글로벌 네트워크 시각화 툴 표준에 근접 (색상 우선 구분)

---

## [5.2.3] - 2026-03-03

### 🐛 버그 수정 (Bug Fixes)

- **Services 헤더 서비스 수 이중집계 수정** (`ServiceListV3.tsx`):
  - Epipe는 2개 장비에 동일 서비스 ID가 존재해 `filteredServices` 기반 집계 시 2배로 카운트되던 문제 수정
  - `filteredServices.reduce()` → 이미 고유 서비스 ID로 그룹화된 변수 합산으로 변경
  - `epipeServices.length + vplsServices.length + vprnServices.length + iesInterfaceCount`
  - Services 헤더 수치(616)가 대시보드 통계 카드와 일치

---

## [5.2.2] - 2026-03-03

### 🐛 버그 수정 (Bug Fixes)

- **대시보드 서비스 카운트 이중집계 수정** (`Dashboard.tsx`):
  - Epipe는 두 사이트를 연결하는 P2P 서비스로 각 사이트에서 한 번씩 집계되어 2배로 카운트되던 문제 수정
  - `siteGroups.serviceCounts` 합산 방식 → `configs` 기반 고유 서비스 ID 집계 방식으로 변경
  - 대시보드 통계 카드의 Epipe/VPLS 카운트가 Services 패널 카운트와 일치

### 🎨 UI/UX 개선 (UI/UX Improvements)

- **Services 헤더 개선** (`ServiceListV3.tsx`):
  - "Network Services" → "Services"로 간소화
  - 카운트 표시: 선택 항목 있을 때 선택 수, 없을 때 전체 서비스 수 표시
  - IES는 인터페이스 단위(= 고객 서비스 단위)로 정확하게 집계

- **그룹 기본값 접힘으로 변경** (`ServiceListV3.tsx`):
  - 초기 로딩 시 Epipe/VPLS/VPRN/IES 그룹 모두 접힌 상태로 시작
  - 검색어 없을 때 그룹 강제 펼침 버그 수정 (searchQuery useEffect else 브랜치 제거)

- **그룹 헤더 텍스트 간소화** (`ServiceListV3.tsx`):
  - "VPLS Services" → "VPLS", "VPRN Services" → "VPRN", "IES Services" → "IES"

---

## [5.2.1] - 2026-03-02

### 🎨 UI/UX 개선 (UI/UX Improvements)

- **Type 버튼 통합 선택/해제** (`ServiceListV3.tsx`):
  - `Select: 전체 / 해제` 버튼 영역 완전 제거
  - `Type:` 라벨 제거
  - Type 필터 버튼이 전체선택/해제 토글 기능 통합
    - **1회 클릭 (다른 타입)**: 필터 전환 + 전체 선택 → 다이어그램 즉시 표시
    - **재클릭 (같은 타입, 선택 있음)**: 전체 해제 → 다이어그램 사라짐
    - **재클릭 (같은 타입, 선택 없음)**: 전체 재선택 → 다이어그램 표시
  - 클릭 수 감소: 타입 변경 + 전체선택 2번 → 1번으로 단축
  - `pendingSelectAll` ref 패턴으로 필터 변경 후 자동선택 구현 (검색 변경과 구분)

---

## [5.2.0] - 2026-03-02

### ✨ 새로운 기능 (New Features)

- **Epipe 현행화 상태 표시** (`ServiceListV3.tsx`):
  - Epipe는 반드시 2개 장비에 설정되어야 정상. 1개 장비만 설정된 경우 "현행화 필요" 표시
  - Epipe 그룹 헤더에 `✅ N` (정상), `⚠️ N` (비정상) 카운트 뱃지 표시
  - 비정상 Epipe 카드에 오렌지 왼쪽 보더 + `⚠️ 현행화 필요 (N개 장비)` 뱃지

- **VPRN/IES HA 뱃지** (`ServiceListV3.tsx`):
  - VPRN 서비스: 정적 라우트 분석으로 HA 감지 → 서비스 헤더에 `HA` 뱃지 표시
  - IES 인터페이스: 동일 prefix에 2개 next-hop 존재 시 인터페이스별 `HA` 뱃지 표시

### 🎨 UI/UX 개선 (UI/UX Improvements)

- **VPLS 다이어그램 `(L2)` SAP 필터링** (`mermaidGeneratorV3.ts`):
  - Nokia 명명 규칙 `To_<hostname>(L2)` 패턴의 SAP는 장비 간 백홀 연결이므로 구성도에서 제외
  - SAP `description` 및 포트 `portDescription` 양쪽 모두 검사하여 `(L2)` 종료 시 필터링
  - 고객 facing SAP만 표시되어 구성도 가독성 향상

- **Epipe HA 뱃지 제거** (`ServiceListV3.tsx`):
  - Epipe는 2개 장비가 1개 서비스를 구성하는 것으로 HA가 아님 (HA = 1461+1462처럼 별도 서비스 ID 쌍)
  - HA 뱃지 대신 현행화 상태(정상/비정상)로 표시 방식 변경

- **v5.1 대시보드 레이아웃 개선** (`Dashboard.tsx`, `V3Page.tsx`):
  - 통계 카드 항상 4열 1줄, 브라우저 스크롤 기반으로 전환
  - 전체 너비 활용, 좁은 화면(모바일) 대응
  - 헤더 버튼 좁은 화면에서 항상 표시
  - 앱 제목 반응형 개선: 640px 이상 풀네임 표시

### 🐛 버그 수정 (Bug Fixes)

- **이중화 필터 자동 선택 오류** (`ServiceListV3.tsx`): `filterType === 'ha'` 시 전체 서비스(`services`) 대신 검색 필터 적용 결과(`filteredServices`)를 기반으로 자동 선택하도록 수정
- **전체 선택 중복 키 오류** (`ServiceListV3.tsx`): `handleSelectAll`에서 동일 serviceId가 2개 config에 존재할 때 중복 키 생성되던 문제 → `Array.from(new Set(allKeys))`로 중복 제거
- **TypeScript 빌드 오류** (`ServiceListV3.tsx`): `filterType === 'ha'`가 `sendChatMessage` 파라미터 타입에 미할당 오류 → `filterType === 'ha' ? 'all' : filterType`으로 처리
- **`/api/config` Rate Limit 제외** (`server/`): 글로벌 Rate Limit에서 config 업로드 경로 제외하여 대용량 배치 업로드 가능하도록 수정

---

## [5.0.0] - 2026-03-01

### 🎯 주요 변경 (Breaking Changes)
- **V1 레거시 제거**: V1Page, nokiaParser, TopologyEngine 등 7개 파일 삭제. `/v1` 경로 제거
- **Vanilla CSS 제거**: 11개 CSS 파일(~4,800줄) 삭제, Tailwind CSS utility class로 전면 마이그레이션

### ✨ 새로운 기능 (New Features)
- **사이트별 대시보드**: Config 로드 후 첫 화면으로 서비스 통계 카드 + 사이트 카드 그리드 표시
  - Hostname에서 사이트명 자동 추출 (siteGrouper 유틸리티)
  - HA 페어 자동 감지 (2대 이상 장비 사이트)
  - 사이트 카드 클릭 → 해당 사이트 서비스 자동 선택 + Services 뷰 전환
  - 사이트명/hostname 검색 필터
- **다크모드**: Sun/Moon 토글, localStorage 저장, prefers-color-scheme 자동 감지
  - NOC/관제센터 야간 근무 최적화
  - 다이어그램 영역은 bg-white 유지 (Mermaid SVG 호환)
- **Dashboard / Services 모드 토글**: 헤더에서 원클릭 전환

### 🎨 UI/UX 개선 (UI/UX Improvements)
- **Tailwind CSS v4 도입**: @tailwindcss/vite 플러그인 기반, 모든 컴포넌트 utility class 적용
- **다크모드 지원**: 전체 UI에 dark: 변형 적용 (배경, 텍스트, 보더)
- **Mermaid QoS CSS 분리**: src/styles/mermaid-overrides.css로 분리 (Tailwind 변환 불가 영역)

### 🔒 보안 강화 (Security Hardening)
- **CORS 제한**: `*` → `http://localhost:3301` (환경변수로 설정 가능)
- **API Key 인증**: X-API-Key 헤더 기반 미들웨어 (server/src/middleware/auth.ts)
- **Path Traversal 방어**: watch-folder 경로 화이트리스트 + 심볼릭 링크 차단
- **WebSocket Origin 검증**: verifyClient 콜백으로 origin 검증
- **글로벌 API Rate Limiting**: /api 전체에 90 req/min 제한 추가
- **Health 정보 은닉**: region/model 정보 제거, `{"status":"ok"}` 만 반환
- **에러 응답 정리**: 내부 에러 메시지 제거, 제네릭 메시지로 통일
- **CSP 헤더**: nginx.conf에 Content-Security-Policy + Referrer-Policy 추가

### ⚡ 성능 최적화 (Performance Optimization)
- **Config 로딩 배치화**: 89개 파일 동시 fetch → 10개씩 배치 fetch
- **@tanstack/react-virtual 설치**: 서비스 리스트 가상화 준비

### 🐳 Docker 변경 (Docker Changes)
- **docker-compose.yml**: CORS_ORIGIN, API_KEY 환경변수 추가
- **nginx.conf**: CSP 헤더, Referrer-Policy 보안 헤더 추가

### 📦 의존성 변경 (Dependencies)
- **추가**: tailwindcss, @tailwindcss/vite, @tanstack/react-virtual
- **제거**: 11개 Vanilla CSS 파일

### 📝 파일 변경 요약
- **삭제**: 18개 파일 (V1 7개 + CSS 11개)
- **신규**: 6개 파일 (Dashboard, useDarkMode, siteGrouper, site.ts, mermaid-overrides.css, auth.ts)
- **수정**: 19개 파일 (Tailwind 변환, 보안 강화, Docker 설정)
- **총 변경**: +2,442줄 / -6,126줄 (순 3,684줄 감소)

---

## [4.8.2] - 2026-02-24

### 🐛 버그 수정

- **Copy PNG Fallback** (`src/components/v3/ServiceDiagram.tsx`): HTTP 환경에서 `navigator.clipboard.write()` API가 Secure Context 제한으로 실패하던 문제 수정. Clipboard API 실패 시 PNG 파일 자동 다운로드로 fallback 처리하여 모든 환경에서 정상 동작

## [4.8.1] - 2026-02-23

### 🔧 원격 접속성 개선 (Remote Connectivity Fixes)

- **Nginx 설정**: `/ws` 경로에 대한 웹소켓 프록시 설정을 추가하여 원격 클라이언트와의 양방향 통신 지원
- **프론트엔드**: 하드코딩된 `localhost:3001` 참조 제거. `window.location.host` 및 상대 경로를 사용하도록 수정하여 원격 접속 환경 최적화

## [4.8.0] - 2026-02-21

### 🤖 NCV AI Collaboration Platform

Nokia Config Visualizer를 AI 협업 미들웨어로 확장하는 플랫폼 기능을 추가했습니다.

#### Feature 1: Structured JSON Output API

- **ConfigStore** (`server/src/services/configStore.ts`): 파싱된 Config 결과를 In-Memory Map으로 관리하는 싱글톤 저장소
- **NCV REST API** (`server/src/routes/ncv.ts`): 8개 엔드포인트 신규 추가
  - `POST /api/ncv/analyze` — 프론트엔드 파싱 결과 백엔드 동기화
  - `GET /api/ncv/services` — 서비스 목록 (type/hostname/q 필터)
  - `GET /api/ncv/services/:serviceKey` — 서비스 상세 조회
  - `GET /api/ncv/topology` — 토폴로지 (JSON/Mermaid)
  - `GET /api/ncv/devices` — 장비 목록
  - `GET /api/ncv/search` — 키워드 검색
  - `GET /api/ncv/export` — 전체 데이터 내보내기 (JSON/Mermaid)
  - `GET /api/ncv/stats` — 시스템 통계
- **useConfigSync** (`src/hooks/useConfigSync.ts`): Config 로드 시 백엔드 자동 동기화 React 훅
- **V3Page 통합**: `useConfigSync(parsedConfigs)` 1줄 추가로 자동 활성화

#### Feature 2: MCP Server (Model Context Protocol)

- **MCP 도구 정의** (`server/src/services/mcpTools.ts`): 7개 도구 스펙 (get_services, get_service_detail, get_topology, search_config, get_devices, get_stats, get_ha_pairs)
- **stdio MCP 서버** (`server/src/mcp-server.ts`): Claude Desktop, Cursor 등에서 직접 연결 가능한 독립 실행 서버
- **HTTP MCP 엔드포인트** (`POST/GET /mcp`): 웹 기반 AI 에이전트용 HTTP transport

#### Feature 3: Semantic Search (RAG Indexing)

- **청크 빌더** (`server/src/services/chunkBuilder.ts`): 서비스 데이터 → 자연어 텍스트 청크 변환
- **임베딩 서비스** (`server/src/services/embeddingService.ts`): Amazon Titan Embed Text v2 (Bedrock) 기반 벡터 생성
- **RAG 인덱서** (`server/src/services/ragIndexer.ts`): vectra 로컬 파일 기반 벡터 DB
- **RAG 엔드포인트**:
  - `POST /api/ncv/index` — 벡터 인덱스 빌드 트리거
  - `POST /api/ncv/semantic-search` — 자연어 시맨틱 검색
  - `GET /api/ncv/index/status` — 인덱스 상태 조회
- **AI 챗봇 개선**: `claudeClient.ts`에 RAG 컨텍스트 자동 주입 파라미터 추가

#### 설정 & 인프라

- **환경변수** (`server/src/config.ts`): RAG 관련 설정 추가 (`RAG_INDEX_PATH`, `EMBEDDING_MODEL`)
- **Docker** (`server/Dockerfile`, `docker-compose.yml`): RAG 인덱스 디렉토리 생성 및 환경변수 추가
- **패키지** (`server/package.json`): `@modelcontextprotocol/sdk@^1.15.0`, `vectra@^0.9.0` 추가

#### UI 변경

- 앱 헤더 이름: `AI Visualizer` → `AI Platform Visualizer`

---

## [4.7.5] - 2026-02-21

### ⚡ 성능 최적화 (Performance Optimization)

Vercel React Best Practices 기반으로 추가 코드 개선 작업을 진행했습니다.

#### 🚨 CRITICAL - 번들 크기 최적화

- **`bundle-dynamic-imports`**: 라우트 레벨 코드 스플리팅 적용
  - `App.tsx`: V3Page, V1Page를 `React.lazy()` + `<Suspense>` 로 변환
  - 초기 번들 ~420KB 추가 절감 (라우트별 지연 로딩)
  - `App.css`: `.page-loading` Suspense 폴백 스타일 추가

#### 🔧 MEDIUM - 리렌더링 최적화

- **`rerender-memo`**: 순수 컴포넌트에 `React.memo()` 적용
  - `ServiceDiagram`: mermaid 재렌더링 방지 (검색/필터 상태 변경 시)
  - `AliasBadge`: 리스트 내 다수 인스턴스 불필요한 재렌더 방지
  - `ConfigFileList`: 파일 목록 / activeFiles 불변 시 재렌더링 방지

- **`rerender-dependencies`** (useLatest 패턴): `V3Page.tsx` stale closure 해결
  - `config-file-changed` 이벤트 핸들러: `activeFiles` 의존성 → `activeFilesRef.current` 참조
  - `useEffect` 의존성 배열 `[activeFiles]` → `[]` (마운트 1회만 등록)

#### 🔎 LOW-MEDIUM - JS 성능 최적화

- **`js-tosorted-immutable`**: 원본 배열 변이 방지를 위해 `Array.sort()` → `Array.toSorted()` 전환 (ES2023)
  - `ServiceListV3.tsx`: IP 정렬(342), serviceId 정렬(643), HA pair 정렬(841)
  - `TopologyEngine.ts`: Static Route HA 정렬(57), VRRP 인터페이스 정렬(104)
  - `DictionaryEditor.tsx`: sortedEntries useMemo 내 정렬(76)

- **`js-hoist-regexp`**: 루프 내 RegExp 재생성 방지 — 모듈 레벨 상수로 호이스팅
  - `parserV3.ts`: 43개 고정 패턴을 `RE_*` 모듈 상수로 호이스팅
  - 영향 함수: `parseQosPolicyDefinitions`, `parseSAPs`, `parseSpokeSDP`, `parseMeshSDP`, `parseEpipe`, `parseVPLS`, `parseVPRN`, `parseSDPs`, `parseL2VPNServices`, `extractPortInfo`, `parseL2VPNConfig`
  - 대규모 config 파싱 시 20-30% 성능 개선 기대

---

## [4.7.4] - 2026-02-21

### ⚡ 성능 최적화 (Performance Optimization)

Vercel React Best Practices 기반으로 번들 크기 축소 및 런타임 성능을 대폭 개선했습니다.

#### 🚨 CRITICAL - 번들 크기 최적화 (Bundle Size Optimization)

- **`bundle-barrel-imports`**: lucide-react 배럴 임포트 → 직접 경로 임포트로 변경 (14개 파일)
  - `from 'lucide-react'` → `from 'lucide-react/dist/esm/icons/<icon>'`
  - Tree-shaking 활성화로 사용하지 않는 아이콘 번들에서 완전 제거
  - 수정 파일: `V3Page.tsx`, `V1Page.tsx`, `ServiceListV3.tsx`, `ServiceDiagram.tsx`, `DiagramViewer.tsx`, `AIChatPanel.tsx`, `DictionaryEditor.tsx`, `GrafanaExportModal.tsx`, `ConfigFileList.tsx`, `FolderPathSettings.tsx`, `ConfigSelector.tsx`, `InterfaceList.tsx`, `FileUpload.tsx`, `FilePreviewModal.tsx`

- **`bundle-dynamic-imports`**: 무거운 라이브러리 및 모달 컴포넌트 동적 임포트 적용
  - `ServiceDiagram.tsx`: `import mermaid from 'mermaid'` (정적) → `await import('mermaid')` (동적)
  - `ServiceListV3.tsx`: `AIChatPanel`, `DictionaryEditor` → `React.lazy()` + `<Suspense fallback={null}>` 적용
  - 초기 번들에서 mermaid 및 무거운 모달 컴포넌트 제거

#### 🔧 MEDIUM - 리렌더링 최적화 (Re-render Optimization)

- **`rerender-memo`**: V3Page 렌더링 시마다 재계산되던 파생 값들에 `useMemo` 적용 (10개)
  - `allServices`, `selectedServices`, `remoteDeviceMap`, `serviceGroups`
  - `iesGroupEntries`, `nonIesGroupEntries`, `iesDiagrams`, `nonIesDiagrams`, `diagrams`
  - `ServiceListV3.tsx`: `epipeServices`, `vplsServices`, `vprnServices`, `iesServices` 분류도 `useMemo` 적용

- **`rerender-memo` (useCallback)**: V3Page 이벤트 핸들러에 `useCallback` 적용
  - `handleToggleService`, `handleSetSelected` → `useCallback` 래핑으로 불필요한 하위 컴포넌트 리렌더 방지

- **`rerender-functional-setstate`**: Stale closure 위험 핸들러 개선
  - VPRN `handleServiceSelect`, `handleInterfaceToggle` → `onSetSelected(prev => ...)` functional updater 패턴 적용
  - IES `handleHostSelect`, `handleInterfaceToggle` → 동일한 functional updater 패턴 적용

- **`rendering-hoist-jsx`**: `mermaid.initialize()` 컴포넌트 마운트마다 중복 호출 제거
  - `DiagramViewer.tsx`: `useEffect(fn, [diagrams])` 안에 있던 initialize 로직 → `useEffect(fn, [])` 분리
  - `ServiceDiagram.tsx`: 모듈 레벨 `mermaidInitialized` 플래그로 최초 1회만 초기화

#### 🔎 LOW-MEDIUM - JS 성능 최적화 (JavaScript Performance)

- **`js-index-maps`**: `configs.find()` O(n) 선형 탐색 → Map O(1) 조회로 개선
  - `configByHostname = useMemo(() => new Map(configs.map(c => [c.hostname, c])), [configs])`
  - `iesDiagrams`, `nonIesDiagrams` 생성 시 `configByHostname.get()` 사용

- **`js-set-map-lookups`**: `selectedServiceIds.includes()` O(n) → `Set.has()` O(1) 전환
  - `selectedSet = useMemo(() => new Set(selectedServiceIds), [selectedServiceIds])`
  - `ServiceListV3.tsx` 내 14개 `.includes()` 호출 전체 대체

- **`js-combine-iterations`**: `Object.values(groupedServices).filter(type === X)` 4회 반복 → 단일 패스 처리
  - switch 문 기반 단일 `for...of` 루프로 `epipe/vpls/vprn/ies` 분류 동시 처리

#### 🎨 LOW - CSS 최적화 (Rendering Optimization)

- **`rendering-hoist-jsx`**: 모달 인라인 style 객체 → CSS 클래스 분리 (`V3Page.tsx` / `V3Page.css`)
  - `style={{position: 'fixed', inset: 0, ...}}` → `.modal-overlay` CSS 클래스
  - `style={{background: '#fff', borderRadius: '8px', ...}}` → `.modal-content` CSS 클래스
  - `style={{position: 'absolute', top: '16px', ...}}` → `.modal-close-btn` CSS 클래스
  - 렌더링마다 새 객체 생성 방지 (참조 안정성 확보)

### 🐛 버그 수정 (Bug Fixes)

- **TypeScript TS7016 오류 수정**: lucide-react 직접 경로 임포트에 대한 타입 선언 추가
  - `src/types/lucide-react-paths.d.ts` 생성: 와일드카드 ambient 모듈 선언
  - `declare module 'lucide-react/dist/esm/icons/*'` 패턴 적용
  - Docker 환경 `tsc -b` 빌드 오류 해결 (최상위 import 제거 → declare module 내부로 이동)

---

## [4.7.1] - 2026-02-20

### 🐛 버그 수정 (Bug Fixes)
- **Demo 환경에서 WebSocket 연결 시도 방지**: Cloudflare Pages 같은 정적 사이트에서 불필요한 WebSocket 연결 시도 제거
  - `useConfigWebSocket.ts`: Demo/Beta/Cloudflare 환경 감지 시 WebSocket 연결 비활성화
  - `ConfigFileList.tsx`: Demo 환경에서 연결 상태 badge/text 표시하지 않음
  - `FolderPathSettings.tsx`: Demo 환경에서 기능 제한 경고 메시지 표시
  - 5번 재시도로 인한 불필요한 콘솔 오류 제거
  - 정적 사이트에서는 수동 Upload만 사용 가능

### 🔧 기술적 변경 (Technical Changes)
- **환경 감지 조건**:
  - `window.location.hostname.includes('demo')`
  - `window.location.hostname.includes('beta')`
  - `window.location.hostname.includes('pages.dev')`
  - `window.location.hostname.includes('cloudflare')`
- Demo 환경에서는 WebSocket status가 'disconnected'로 유지되며 연결 시도하지 않음

---

## [4.7.0] - 2026-02-20

### ✨ 새로운 기능 (New Features)
- **자동 Config 로딩 (Auto Config Loading)**: 로컬 폴더 자동 감시를 통한 Config 파일 자동 로딩
  - **간단한 사용법**: `./configs` 폴더에 파일 복사만 하면 자동 로딩
  - Docker volume mount로 백업 폴더 연결 (`./configs:/app/configs:ro`)
  - 파일 추가/변경/삭제 시 실시간 자동 감지 (chokidar v4.0.1)
  - WebSocket으로 프론트엔드에 즉시 알림 (ws v8.18.0)
  - hostname별 최신 파일만 자동 필터링
  - **다양한 파일명 형식 지원**:
    - `router1-20260219.txt` (하이픈 + YYYYMMDD)
    - `router1_20260219.txt` (언더스코어 + YYYYMMDD)
    - `router1 20260219.txt` (공백 + YYYYMMDD)
    - `router1-2026-02-19.txt` (하이픈 + YYYY-MM-DD)
    - `router1_2026_02_19.txt` (언더스코어 + YYYY_MM_DD)
  - **멀티파일 선택**: 모든 파일 기본 활성화, 클릭으로 토글 (추가/제거)
  - **접을 수 있는 사이드바**: Config 파일 목록을 접어 화면 공간 확보 (기본: 접힘)
  - 기존 수동 업로드와 병행 사용 가능

### 🎨 UI/UX 개선 (UI/UX Improvements)
- **일관된 토글 버튼 디자인**:
  - Config 토글: `FolderOpen`/`Folder` 아이콘 (좌측 사이드바 제어)
  - Services 토글: `PanelLeft`/`PanelLeftClose` 아이콘 (우측 사이드바 제어)
  - 통일된 스타일: border, hover, active 상태 일관성
- **헤더 레이아웃 개선**:
  - Config 버튼을 Services 버튼 앞으로 이동 (좌→우 순서: Config, Services)
  - "자동 로딩"과 "Upload" 버튼을 Config 사이드바 하단으로 이동
  - 헤더 제목이 1줄로 표시되도록 공간 확보
- **파일 목록 표시 개선**:
  - hostname + 날짜(YYYY-MM-DD) 형식으로 간결하게 표시
  - 날짜는 monospace 폰트로 정렬
  - 활성 파일은 파란색 호스트명과 ● 표시
- **Collapsible 사이드바**:
  - 토글 버튼으로 Config 목록 접기/펼치기
  - 기본 상태: 접힘 (다이어그램 공간 최대화)
  - 부드러운 0.3s 전환 애니메이션

### 🔧 Backend 변경 (Backend Changes)
- **새 파일**:
  - `server/src/utils/configFilenameParser.ts`: 똑똑한 파일명 파싱 유틸리티
  - `server/src/services/fileWatcher.ts`: File Watcher 서비스 (chokidar 기반)
  - `server/src/services/websocket.ts`: WebSocket 서버 (ws 기반)
  - `server/src/routes/config.ts`: Config 파일 관리 API 엔드포인트
- **수정 파일**:
  - `server/src/index.ts`:
    - HTTP 서버 생성, WebSocket 서버 설정, File Watcher 자동 시작
    - **Rate Limit 제외**: `/api/config` 경로는 Rate Limit 미적용 (대량 파일 로딩 지원)
    - Rate Limit은 `/api/chat`과 `/api/dictionary`에만 적용
  - `server/package.json`: chokidar@4.0.1, ws@8.18.0, @types/ws@8.5.13 추가
- **새 API 엔드포인트**:
  - `GET /api/config/files`: 파일 목록 조회 (최신만/전체)
  - `GET /api/config/file/:filename`: 파일 다운로드
  - `GET /api/config/watch-status`: 감시 상태 조회
  - `GET /api/config/groups`: 파일 그룹 정보 조회 (hostname별)

### 🎨 Frontend 변경 (Frontend Changes)
- **새 파일**:
  - `src/types/configWebSocket.ts`: WebSocket 관련 타입 정의 (멀티파일 지원)
  - `src/hooks/useConfigWebSocket.ts`: WebSocket 연결 관리 Custom Hook
  - `src/components/v3/ConfigFileList.tsx`: Config 파일 목록 컴포넌트 (토글 선택, 하단 버튼)
  - `src/components/v3/ConfigFileList.css`: Config 파일 목록 스타일
  - `src/components/v3/FolderPathSettings.tsx`: 폴더 경로 사용법 안내 컴포넌트
  - `src/components/v3/FolderPathSettings.css`: 폴더 경로 설정 스타일
- **수정 파일**:
  - `src/pages/V3Page.tsx`:
    - useConfigWebSocket Hook 사용
    - ConfigFileList 접을 수 있는 사이드바 추가
    - FolderPathSettings 모달 추가
    - `config-files-load-all`, `config-file-selected`, `config-file-removed` 이벤트 리스너
    - Promise.all로 병렬 파일 로딩
    - Config/Services 버튼 순서 변경
  - `src/pages/V3Page.css`: 일관된 토글 버튼 스타일 (`.sidebar-toggle-btn`)
  - `src/components/FileUpload.tsx`: `compact` variant 추가 (사이드바용)
  - `src/App.css`: `.btn-upload-compact` 스타일 추가

### 🐳 Docker 변경 (Docker Changes)
- **docker-compose.yml**:
  - nokia-api 서비스에 volume mount 추가: `./configs:/app/configs:ro` (고정 경로)
  - **포트 매핑 수정**: `3001:3000` (외부 3001 → 내부 3000)
- **.gitignore**:
  - `configs/*` 추가 (보안상 config 파일 유출 방지)
  - `!configs/.gitkeep` 예외 (폴더 구조 유지)
- **configs/.gitkeep**: 빈 파일 생성 (Git에서 폴더 구조 보존)

### 🐛 버그 수정 (Bug Fixes)
- **포트 매핑 오류 수정**: docker-compose.yml에서 `3001:3001` → `3001:3000`으로 수정
  - WebSocket 연결 실패 (429 오류) 해결
- **Rate Limit 429 오류 수정**: 85개 파일 동시 로딩 시 발생하던 429 오류 해결
  - `/api/config` 경로를 Rate Limit에서 제외
  - AI 챗봇(`/api/chat`)과 사전(`/api/dictionary`)에만 Rate Limit 적용
- **TypeScript 브라우저 호환성**: `NodeJS.Timeout` → `number`로 수정
- **단일 파일만 로딩되던 문제 수정**: `activeFile` → `activeFiles` 배열로 변경
  - 모든 파일 기본 활성화
  - 토글 기능으로 개별 파일 추가/제거 가능

### 📝 문서 업데이트 (Documentation)
- **README.md**:
  - 버전 업데이트: v4.6.5 → v4.7.0
  - "검색 예시 Pills" 기능 설명 추가 (v4.6.5)
  - "자동 Config 로딩" 섹션 업데이트 (v4.7.0 최종 구현 반영)
  - 기술 스택에 chokidar, WebSocket 추가

### v4.8.1 (Next Steps)
- **다음 단계**: Phase 2 - 동적 예시 생성 (v4.8.1 예정)

### 🔄 호환성 (Compatibility)
- **Breaking Changes**: 없음 (기존 수동 업로드 기능 유지)
- **브라우저 호환성**: setTimeout/clearTimeout 타입 수정 (브라우저 환경 완전 호환)

---

## [4.6.1] - 2026-02-19

### 🐛 버그 수정 (Bug Fixes)
- **복수 검색어 Catch-all 검색 강화**: ServiceListV3의 AND/OR 검색 로직에서 `JSON.stringify(service)`를 searchFields에 추가
  - 단일 검색어에서만 작동하던 catch-all 검색이 복수 검색어(AND/OR)에서도 작동하도록 개선
  - 필드명 자체(예: "portId", "portDescription")도 AND/OR 검색 가능
  - 명시적으로 수집하지 못한 필드나 중첩된 객체 내부 값도 검색 가능
  - 예시: `port Video`, `port + Video`, `qos 100M`, `epipe 1270` 모두 정상 작동
- **백엔드 TypeScript 컴파일 오류 수정**: `server/src/routes/config.ts`에서 `path.basename(filename as string)` 타입 캐스팅 추가

### 🔧 기술적 변경 (Technical Changes)
- **수정 파일**:
  - `src/components/v3/ServiceListV3.tsx` (라인 591-600): 복수 검색어 로직에 JSON.stringify 추가
  - `server/src/routes/config.ts` (라인 112): TypeScript 타입 캐스팅 추가

### 📝 검증 결과 (Verification)
- **테스트 항목**: 사용자 피드백 기반 6개 검색 케이스 검증
  1. ✅ 'qos 100M': OR 검색 정상 작동
  2. ✅ 'port Video': OR 검색 정상 작동
  3. ✅ 'port + Video': AND 검색 정상 작동
  4. ✅ 'epipe 1270': OR 검색 정상 작동
  5. ✅ '1/1/1': 단일 검색 정상 작동
  6. ✅ 'vpls': 단일 검색 정상 작동

---

## [4.6.5] - 2026-02-19

### ✨ 새로운 기능 (New Features)
- **검색 예시 UI (Search Examples Pills)**: 검색창 아래에 6개 클릭 가능한 예시 pill 표시
  - QoS, IP 서브넷, AND 검색, 서비스 ID, 포트, 서비스 타입 예시 제공
  - 클릭 시 검색창에 자동 입력, 사용자가 수정 가능
  - Tooltip으로 각 예시 설명 표시 (호버 시 보임)
  - 키보드 접근성 지원 (Tab 네비게이션, Enter/Space로 실행)

### 🎨 UI/UX 개선 (UI/UX Improvements)
- **Hover/Active 효과**: 예시 pill에 시각적 피드백 (색상 변경, 애니메이션)
- **반응형 디자인**: 모바일(< 768px) 및 태블릿(768-1024px)에 최적화된 레이아웃
- **접근성**: aria-label, title 속성으로 스크린 리더 및 마우스 오버 Tooltip 지원

### 🔧 기술적 변경 (Technical Changes)
- **수정 파일**:
  - `src/components/v3/ServiceListV3.tsx`:
    - `SearchExample` 인터페이스 추가 (라인 29-38)
    - `STATIC_EXAMPLES` 배열로 6개 고정 예시 정의 (라인 65-72)
    - `handleExampleClick` useCallback 핸들러 추가 (라인 107-109)
    - 예시 pills JSX 렌더링 (라인 779-796)
  - `src/components/v3/ServiceListV3.css`:
    - `.search-examples-container` 컨테이너 스타일 (라인 278-285)
    - `.examples-label` 라벨 스타일 (라인 290-295)
    - `.examples-pills` pills 컨테이너 (라인 300-304)
    - `.example-pill` 기본/hover/active/focus/disabled 상태 (라인 309-372)
    - 반응형 미디어 쿼리 (라인 381-408)

### 📝 문서화 (Documentation)
- **완성 보고서**: `docs/04-report/features/search-examples-ui.report.md` 작성
  - PDCA 사이클 전체 통합 (Plan → Design → Do → Check → Act)
  - 설계-구현 일치율 95% 달성
  - 테스트 결과 및 성능 지표 기록

### 🎯 Phase 1 완료 정보
- **구현 시간**: ~2.5시간 (Plan → Design → Do → Check)
- **반복 횟수**: 0회 (첫 코드에서 완성)
- **테스트 상태**: 모든 기능 테스트 통과
- **다음 단계**: Phase 2 - 동적 예시 생성 (v4.8.1 예정)

---

## [4.6.0] - 2026-02-19

### 🚀 주요 기능 (Major Features)
- **IP 서브넷 검색 (IP Subnet Search)**: 검색창에서 IP 주소로 네트워크 서브넷 검색 기능 추가
  - IPv4 주소 입력 시 자동으로 IP 검색 모드 활성화
  - Static Route 설정 기반 서브넷 매칭 (예: `10.0.1.50` 검색 → `10.0.1.0/24` 서브넷 매칭)
  - Longest Prefix Match (LPM) 알고리즘으로 가장 구체적인 서브넷 우선 정렬
  - IES 서비스의 인터페이스 레벨 필터링 (검색 IP와 관련된 인터페이스만 표시)

### ✨ 새로운 기능 (New Features)
- **IPv4 유효성 검증**: 정규식 기반 IP 주소 형식 검증 및 옥텟 범위 체크 (0-255)
- **서브넷 계산**: CIDR 표기법 파싱 및 비트 연산을 통한 네트워크 주소 계산
- **너무 넓은 서브넷 필터링**: `/8` 미만의 과도하게 넓은 서브넷 자동 제외 (예: `128.0.0.0/1` 제외)
- **다중 호스트 지원**: hostname 정보를 보존하여 동일 serviceId의 서로 다른 호스트 정확히 구분
- **인터페이스 레벨 필터링**: IES 서비스에서 검색 IP와 관련된 Static Route를 가진 인터페이스만 표시

### 📦 신규 모듈 (New Modules)
- **`src/utils/ipUtils.ts`**: IP 유틸리티 함수 모듈 추가
  - `isValidIPv4()`: IPv4 주소 유효성 검증
  - `ipToLong()`: IP 주소를 32비트 정수로 변환
  - `parseNetwork()`: CIDR 표기법 파싱 (예: "10.0.1.0/24")
  - `isIpInSubnet()`: IP 주소가 서브넷에 포함되는지 확인
  - `sortByLongestPrefix()`: Longest Prefix Match 정렬

### 🔧 기술적 변경 (Technical Changes)
- **수정 파일**:
  - `src/components/v3/ServiceListV3.tsx`:
    - IP 검색 모드 추가 (라인 154-260)
    - `matchServiceByIpSubnet()` 함수로 서브넷 매칭 (라인 118-149)
    - IES 인터페이스 레벨 필터링 로직 (라인 202-260)
    - Longest Prefix Match 정렬 알고리즘 (라인 186-197)
  - `package.json`: 버전 4.5.6 → 4.6.0

### 📊 검색 알고리즘 (Search Algorithm)
1. **IPv4 검증**: `/^(\d{1,3}\.){3}\d{1,3}$/` 정규식 + 각 옥텟 0-255 범위 확인
2. **서비스 레벨 매칭**: 모든 서비스의 Static Routes에서 검색 IP가 포함된 서브넷 찾기
3. **LPM 정렬**: prefixLen 내림차순 (큰 값 = 더 구체적인 매칭)
4. **인터페이스 필터링** (IES만): V1 변환 후 `findPeerAndRoutes()`로 관련 라우트 확인
5. **결과 반환**: 검색 IP와 관련된 서비스 및 인터페이스만 표시

### 💡 사용 예시 (Usage Examples)
