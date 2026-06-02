# Release Notes Index - Nokia Config Visualizer

## 최신 버전

| 버전 | 날짜 | 타입 | 주요 내용 |
|------|------|------|----------|
| [v5.8.8](v5.8.8.md) | 2026-06-02 | Bug Fix | 검색 필터 버그 수정 — SAP/SDP description 누락, IES hostname 검색 불가, Unicode 정규화 불일치 |
| [v5.8.7](v5.8.7.md) | 2026-04-22 | Infrastructure | 포털 전용 접속 전환 — Authentik 인증 제거, VITE_API_URL 동적화 |
| [v5.8.5](v5.8.5.md) | 2026-04-10 | UX | "파일에서도 발견됨" 섹션 — 미로드 파일만 표시 (로드된 파일 중복 제거) |
| [v5.8.4](v5.8.4.md) | 2026-04-10 | Bug Fix | extractSection 빈 service 블록 건너뜀 — 7750SR-a8 파일 서비스 미파싱 버그 수정 |
| [v5.8.3](v5.8.3.md) | 2026-04-10 | Feature | watchFolder 전체 파일 텍스트 검색 (search-global-config) — 미로드 파일 발견 + 즉시 로드 |
| [v5.8.2](v5.8.2.md) | 2026-04-10 | Enhancement | Dashboard 카드 클릭 시 해당 장비 서비스 필터링 (사이드바 배너 + X 해제) |
| [v5.8.1](v5.8.1.md) | 2026-04-10 | Performance | React 성능 최적화 v2 (groupedDiagrams useMemo, handleCopyCode/handleZoom useCallback, Dashboard 상수화) |
| [v5.8.0](v5.8.0.md) | 2026-04-10 | Performance | React 성능 최적화 (메모리 누수, Stale Closure, 정규식 최적화, useCallback/useMemo) |
| [v5.7.5](v5.7.5.md) | 2026-04-10 | Bug Fix | IES QoS 엣지 레이블 Copy PNG 렌더링 수정 (초록 배지 스타일 누락) |
| [v5.7.4](v5.7.4.md) | 2026-04-10 | Enhancement | Mermaid flowchart LR 전환, 노드 포맷 개편, Hostname 줄바꿈 방지 |
| [v5.7.3](v5.7.3.md) | 2026-04-10 | Enhancement | Show Code → Copy Code 버튼 변환 (1단계 클립보드 복사) |
| [v5.7.2](v5.7.2.md) | 2026-04-08 | Bug Fix | Mermaid 다이어그램 silent failure 수정 |
| [v5.7.1](v5.7.1.md) | 2026-04-03 | Bug Fix | 포털 iframe Nokia Visualizer Authentik 인증 흐름 수정 (TOP 프레임 리다이렉트) |
| [v5.7.0](v5.7.0.md) | 2026-04-02 | Security | Authentik SSO Forward Auth 인증 게이트, iframe 허용, 쿠키 도메인 공유 |
| [v5.6.3](v5.6.3.md) | 2026-03-31 | Bug Fix | Mermaid HTML 속성 따옴표 충돌, 노드 ID 하이픈 파싱 오류 수정 |
| [v5.6.2](v5.6.2.md) | 2026-03-21 | Bug Fix | Dashboard 검색 Unicode 하이픈(U+2011) 정규화 버그 수정 |
| [v5.6.1](v5.6.1.md) | 2026-03-19 | Bug Fix | VPLS 4073 ISP/MPLS 다이어그램 분리, networkType 덮어쓰기 버그 수정 |
| [v5.6.0](v5.6.0.md) | 2026-03-19 | Feature | Network Type Separation Phase 1 (ISP/MPLS/Cloud 망 자동 구분) |
| [v5.5.2](v5.5.2.md) | 2026-03-19 | Bug Fix | Unicode 하이픈 문자 검색 오류 수정 (U+2011 Non-Breaking Hyphen) |
| [v5.5.1](v5.5.1.md) | 2026-03-16 | Bug Fix | Copy PNG 클립보드 복사 기능 수정 (NPM Permissions Policy) |
| [v5.5.0](v5.5.0.md) | 2026-03-13 | Feature | AutoParser Service (자동 파싱), Vendor Detection 개선 |
| [v5.4.0](v5.4.0.md) | 2026-03-11 | Feature | v4 AI Platform + v5 UI/Security 통합 |
| [v5.3.1](v5.3.1.md) | 2026-03-11 | Feature | CORS 다중 origin 지원, 자동 업데이트 스크립트 |
| [v5.3.0](v5.3.0.md) | 2026-03-09 | Feature | 폴더 선택 기능 (webkitdirectory, localStorage 유지) |

## v5.2.x 릴리즈

| 버전 | 날짜 | 타입 | 주요 내용 |
|------|------|------|----------|
| [v5.2.6](v5.2.6.md) | 2026-03-03 | Feature | 대시보드 전체 서비스 수 표시 추가 |
| [v5.2.5](v5.2.5.md) | 2026-03-03 | Bug Fix + UI | 이중 스크롤바 제거, 그룹 헤더 Sticky 고정 |
| [v5.2.4](v5.2.4.md) | 2026-03-03 | UI/UX | 서비스 타입별 Visual Identity 통합 |
| [v5.2.3](v5.2.3.md) | 2026-03-03 | Bug Fix | Services 헤더 서비스 수 이중집계 수정 |
| [v5.2.2](v5.2.2.md) | 2026-03-03 | Bug Fix + UI | 대시보드 Epipe 이중집계 수정, Services UI 개선 |
| [v5.2.1](v5.2.1.md) | 2026-03-02 | UI/UX | Type 버튼 통합 선택/해제 |
| [v5.2.0](v5.2.0.md) | 2026-03-02 | Feature | Epipe 현행화 상태, VPRN/IES HA 뱃지, VPLS (L2) SAP 필터링 |

## v5.0.x 릴리즈

| 버전 | 날짜 | 타입 | 주요 내용 |
|------|------|------|----------|
| [v5.0.0](v5.0.0.md) | 2026-03-01 | Major | 사이트별 대시보드, 다크모드, Tailwind CSS v4, V1 제거, 보안 강화 |

## v4.x 릴리즈

| 버전 | 날짜 | 타입 | 주요 내용 |
|------|------|------|----------|
| [v4.8.2](v4.8.2.md) | 2026-02-24 | Bug Fix | Copy PNG HTTP 환경 fallback 추가 |
| [v4.8.1](v4.8.1.md) | 2026-02-23 | Bug Fix | 원격 접속 WebSocket 수정 |
| [v4.8.0](v4.8.0.md) | 2026-02-21 | Feature | NCV AI Collaboration Platform (REST API, MCP, RAG) |
| [v4.7.5](v4.7.5.md) | 2026-02-21 | Performance | 라우트 레벨 코드 스플리팅, React.memo 추가 |
| [v4.7.4](v4.7.4.md) | 2026-02-21 | Performance | 번들 크기 최적화, 리렌더링 최적화, 정규식 호이스팅 |
| [v4.7.1](v4.7.1.md) | 2026-02-20 | Bug Fix | Demo 환경 WebSocket 연결 방지 |
| [v4.7.0](v4.7.0.md) | 2026-02-20 | Feature | 자동 Config 로딩 (FileWatcher + WebSocket) |
| [v4.6.5](v4.6.5.md) | 2026-02-19 | Feature | 검색 예시 Pills UI 추가 |
| [v4.6.1](v4.6.1.md) | 2026-02-19 | Bug Fix | 복수 검색어 catch-all 검색 수정 |
| [v4.6.0](v4.6.0.md) | 2026-02-19 | Feature | IP 서브넷 검색 (LPM 알고리즘) |

## 이전 버전

| 버전 | 주요 내용 |
|------|----------|
| [v4.4.0](v4.4.0.md) | — |
| [v4.1.0](v4.1.0.md) | — |
| [v3.2.0](v3.2.0.md) | — |
| [v3.1.0](v3.1.0.md) | — |
| [v1.8.0](v1.8.0.md) | — |
| [v1.7.0](v1.7.0.md) | — |
| [v1.6.0](v1.6.0.md) | — |
| [v1.5.0](v1.5.0.md) | — |
| [v1.4.0](v1.4.0.md) | — |
