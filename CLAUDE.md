# CLAUDE.md - Nokia Config Visualizer Project Context (v5.6.2)

> 프로젝트 특화 컨텍스트. 전역 지침: `~/.claude/CLAUDE.md` 참조.

## 프로젝트 개요

Nokia 장비 설정 파일을 파싱하여 네트워크 토폴로지를 시각화하는 웹 애플리케이션.

- **Frontend**: React 19 + TypeScript + Tailwind CSS v4
- **Backend**: Express + AWS Bedrock (Claude AI)
- **Build**: Vite (`@tailwindcss/vite` 플러그인)
- **Visualization**: Mermaid.js (Grafana Diagram 패널 호환)
- **Icons**: Lucide React (직접 경로 import: `lucide-react/dist/esm/icons/<icon>`)
- **Infra**: Docker Compose (nginx + Express 별도 컨테이너)

### 데이터 플로우

```
Config Upload → ParserV3 → ParsedConfigV3 → Selection (수동/AI) → Mermaid Generator → Diagram
```

## 핵심 구조

### 주요 디렉토리

- `src/pages/` : V3Page (유일한 메인 페이지)
- `src/components/v3/` : Dashboard, ServiceListV3, ServiceDiagram, AIChatPanel, DictionaryEditor 등
- `src/utils/` : Parser, Mermaid Generator, Adapter, IP Utils, Site Grouper
- `src/utils/v3/` : V3 전용 파서(`parserV3.ts`) 및 다이어그램(`mermaidGeneratorV3.ts`)
- `src/hooks/` : useDarkMode, useConfigSync, useConfigWebSocket
- `src/services/` : chatApi, dictionaryApi (백엔드 API 클라이언트)
- `src/types/` : TypeScript 타입 정의 (services, site, dictionary, grafana 등)
- `server/src/routes/` : chat, config, dictionary, ncv (REST API)
- `server/src/services/` : claudeClient, fileWatcher, websocket, configStore, ragIndexer 등
- `server/src/middleware/` : auth.ts (API Key 인증)

### 주요 파일

| 파일 | 역할 |
|------|------|
| `src/utils/v3/parserV3.ts` | 통합 파서 (Epipe, VPLS, VPRN, IES, BGP, OSPF). 들여쓰기 기반 블록 파싱 |
| `src/utils/v3/mermaidGeneratorV3.ts` | 서비스 중심 다이어그램 (QoS, 라우팅 노드, Grafana 호환) |
| `src/utils/mermaidGenerator.ts` | IES/물리 토폴로지 다이어그램 (Single/HA) |
| `src/utils/v1IESAdapter.ts` | IES 서비스 → v1 형식 변환 |
| `src/utils/v1VPRNAdapter.ts` | VPRN 서비스 → v1 형식 변환 |
| `src/utils/siteGrouper.ts` | Hostname에서 사이트명 자동 추출, HA 페어 감지 |
| `src/utils/configSummaryBuilder.ts` | ParsedConfigV3 → AI용 축약 JSON 변환 |
| `src/components/v3/Dashboard.tsx` | 사이트별 대시보드 (통계 카드 + 사이트 카드 그리드) |
| `server/src/services/claudeClient.ts` | AWS Bedrock Converse API 호출 |
| `server/src/services/dictionaryGenerator.ts` | AI 이름 사전 자동 생성 |
| `server/src/config.ts` | 환경변수 중앙 관리 (AWS, Bedrock, Rate Limit, RAG) |

## 주요 기능 구현 위치

### Config 파싱
- `src/utils/v3/parserV3.ts` → `parseNokiaConfigV3()`

### 다이어그램 생성
- **Epipe/VPLS/VPRN**: `src/utils/v3/mermaidGeneratorV3.ts`
- **IES/물리**: `src/utils/mermaidGenerator.ts` (v1IESAdapter 경유)

### 대시보드 (v5.0)
- `src/components/v3/Dashboard.tsx` + `src/utils/siteGrouper.ts`

### 다크모드 (v5.0)
- `src/hooks/useDarkMode.ts` (localStorage + prefers-color-scheme)

### 검색
- **수동**: `ServiceListV3.tsx` (AND: ` + `, OR: 띄어쓰기, IP 서브넷 LPM)
- **AI**: `AIChatPanel.tsx` → `chatApi.ts` → `claudeClient.ts` → AWS Bedrock

### 자동 Config 로딩
- `server/src/services/fileWatcher.ts` (chokidar) + `websocket.ts` (ws)
- `src/hooks/useConfigWebSocket.ts` (프론트엔드 연결)

### NCV AI Platform (v4.8)
- REST API: `server/src/routes/ncv.ts` (8개 엔드포인트)
- MCP Server: `server/src/mcp-server.ts` (stdio/HTTP)
- RAG: `chunkBuilder.ts` → `embeddingService.ts` → `ragIndexer.ts`

## 개발 가이드

### 코드 작성 규칙
1. **TypeScript Strict**: `any` 사용 금지, 상세 인터페이스 정의
2. **타입 정의**: `src/types/` 디렉토리에 분리 관리
3. **컴포넌트**: React 함수형 컴포넌트 + Hooks
4. **스타일**: Tailwind CSS v4 utility classes (Vanilla CSS 사용 안 함)
   - Mermaid SVG 오버라이드만 `src/styles/mermaid-overrides.css`에 별도 관리
5. **모듈화**: 파싱 로직(`utils/`)과 UI(`components/`) 분리
6. **빌드 제약**: 프론트엔드는 정적 자산만 생성. AI 기능은 별도 Express 백엔드
7. **아이콘**: `lucide-react/dist/esm/icons/<icon>` 직접 경로 import (배럴 import 금지)
8. **성능**: `useMemo`/`useCallback` 적극 활용, `React.lazy()` 동적 import, `Set`/`Map` 사용

### 디자인 & UX 원칙
- 다크모드 지원: 모든 UI에 `dark:` 변형 적용
- 다이어그램 영역: 다크모드에서도 `bg-white` 유지 (Mermaid SVG 호환)
- 데스크톱 우선 (복잡한 다이어그램 대상)

### 파서 작성 시 주의사항
Nokia Config는 들여쓰기 기반 구조. 정규식으로 블록 추출, 들여쓰기 레벨 추적, `exit` 키워드로 블록 종료 판단. 정규식은 모듈 레벨 `RE_*` 상수로 호이스팅.

### Mermaid 코드 생성 시 주의사항
상세 규칙은 `DIAGRAM_RULES.md` 참조. 핵심: 특수문자 이스케이프(`&`, `<`, `>`, `"`), HTML 라벨 Double Quote, CSS class 기반 스타일링.

## 설정 및 버전 관리

### 환경변수 관리

**모든 환경변수는 `server/src/config.ts`에서 중앙 관리.**

우선순위: `docker-compose.yml` > `.env` > `config.ts` 기본값

관리 항목: AWS region/profile, Bedrock modelId, CORS, Rate Limit, API Key, RAG 설정

### 보안 (v5.0)
- CORS 제한, API Key 인증(`X-API-Key`), Path Traversal 방어
- WebSocket Origin 검증, 글로벌 Rate Limiting (90 req/min)
- CSP 헤더 (nginx.conf)

### 버전 관리

**`package.json`의 `version` 필드가 단일 소스.** 빌드 시 `__APP_VERSION__`으로 자동 주입 (`vite.config.ts`).

표시 위치: V3Page 헤더, DictionaryEditor 모달

자동 버전 관리 활성화 시: commit → patch 자동 증가 + tag 생성 → push → Release 자동 생성

Minor/Major 변경 시: hook 비활성화 → `npm run version:minor/major` → 커밋 → tag → hook 재활성화

## 테스트
- 표준 테스트 파일: `public/config1.txt`, `public/config2.txt`

## 참고 문서
- `README.md`: 프로젝트 설명, 빌드/배포 명령어
- `CHANGELOG.md`: 버전별 변경 이력 (요약)
- `docs/releases/`: 버전별 상세 릴리즈 노트
  - `docs/releases/README.md`: 전체 버전 인덱스 테이블
  - `docs/releases/vX.X.X.md`: 버전별 상세 노트 (v1.4.0 ~ v5.6.2, 36개)
- `DIAGRAM_RULES.md`: Mermaid 다이어그램 렌더링 규칙 상세
- `HOWTO-DOCKER.md`: Docker 빌드 및 배포 가이드

### 릴리즈 노트 작성 규칙

- **저장 위치**: `docs/releases/vX.X.X.md` (루트에 `RELEASE_NOTES_*.md` 생성 금지)
- **인덱스 관리**: 새 버전 추가 시 `docs/releases/README.md` 테이블 업데이트
- **헤더 형식**:
  ```markdown
  # Release Notes - vX.X.X

  > **Date**: YYYY-MM-DD
  > **Branch**: `main`
  > **Type**: Feature / Bug Fix / Performance / Major Release
  ```

## 브랜치 전략
- `main`: 프로덕션 코드
- `v5-development`: v5 개발 (현재 활성)

## 최근 버전 히스토리

| 버전 | 날짜 | 주요 내용 |
|---|---|---|
| v5.8.8 | 2026-06-02 | 검색 필터 버그 수정 — SAP/SDP description 누락, IES hostname 검색 불가, Unicode 정규화 불일치 수정 |
| v5.8.5 | 2026-04-10 | "파일에서도 발견됨" 섹션 미로드 파일만 표시 (로드된 파일 중복 제거) |
| v5.8.4 | 2026-04-10 | extractSection 빈 service 블록 건너뜀 — 7750SR-a8 파일 서비스 미파싱 버그 수정 |
| v5.8.1 | 2026-04-10 | React 성능 최적화 v2 (groupedDiagrams useMemo, handleCopyCode/handleZoom useCallback, Dashboard 상수화) |
| v5.8.0 | 2026-04-10 | React 성능 최적화 (메모리 누수, Stale Closure, 정규식 최적화, useCallback/useMemo) |
| v5.7.1 | 2026-04-03 | 포털 iframe Nokia Visualizer Authentik 인증 흐름 수정 (TOP 프레임 리다이렉트) |
| v5.7.0 | 2026-04-02 | Authentik SSO Forward Auth 인증 게이트, iframe 허용, 쿠키 도메인 공유 |
| v5.6.3 | 2026-03-31 | Mermaid HTML 속성 따옴표 충돌, 노드 ID 하이픈 파싱 오류 수정 |
| v5.6.2 | 2026-03-21 | Dashboard 검색 Unicode 하이픈(U+2011) 정규화 버그 수정 |
| v5.6.1 | 2026-03-19 | VPLS 4073 ISP/MPLS 다이어그램 분리, networkType 덮어쓰기 버그 수정 |
| v5.6.0 | 2026-03-19 | Network Type Separation Phase 1 (ISP/MPLS/Cloud 망 자동 구분) |
| v5.5.2 | 2026-03-19 | Unicode 하이픈 문자 검색 오류 수정 (U+2011 Non-Breaking Hyphen) |
| v5.5.1 | 2026-03-16 | Copy PNG 클립보드 복사 기능 수정 (NPM Permissions Policy) |
| v5.5.0 | 2026-03-13 | AutoParser Service (자동 파싱), Vendor Detection 개선 |
| v5.3.0 | 2026-03-09 | 폴더 선택 기능 (webkitdirectory + localStorage 유지, 서브디렉토리 지원) |
| v5.2.6 | 2026-03-03 | 대시보드 전체 서비스 수 표시 추가 |
| v5.2.5 | 2026-03-03 | 이중 스크롤바 제거 + 그룹 헤더 Sticky 고정 |
| v5.2.4 | 2026-03-03 | 서비스 타입별 색상/아이콘 Visual Identity 통합 |
| v5.2.3 | 2026-03-03 | Services 헤더 수치 이중집계 버그 수정 |
| v5.2.2 | 2026-03-03 | 대시보드 Epipe 이중집계 수정, Services UI 개선 (헤더, 그룹 접힘) |
| v5.2.1 | 2026-03-02 | Type 버튼 통합 선택/해제 (Select 영역 제거) |
| v5.2.0 | 2026-03-02 | Epipe 현행화 상태, VPRN/IES HA 뱃지, VPLS (L2) SAP 필터링, 버그 수정 |
| v5.1.0 | 2026-03-01 | 대시보드 반응형 레이아웃, 헤더 반응형 개선, 앱 아이콘 |
| v5.0.0 | 2026-03-01 | 사이트별 대시보드, 다크모드, Tailwind CSS v4, V1 제거, 보안 강화 |

상세 변경 이력은 `CHANGELOG.md` 및 `docs/releases/` 참조.

---

**Last Updated**: 2026-06-02
**Current Version**: v5.8.8
**Branch**: main
