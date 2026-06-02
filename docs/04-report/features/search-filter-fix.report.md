# 완성 보고서: 검색 필터 버그 수정 (search-filter-fix)

> **기능명**: `search-filter-fix` (v5.9.0)
> **작성일**: 2026-06-02
> **상태**: ✅ 완료 (Match Rate: 100%)
> **반복**: 0회 (1차 완성)

---

## 1. 실행 요약 (Executive Summary)

| 관점 | 내용 |
|------|------|
| **Problem** | 단일 검색어에서 Epipe/VPLS SAP·SDP description 누락, IES 서비스를 hostname·serviceId로 검색 불가, Unicode 하이픈 정규화 불일치 |
| **Solution** | 단일 검색어 경로를 복수 경로와 대칭적으로 수정하고, IES 서비스 레벨 매칭(hostname/serviceId/description) 후 인터페이스 필터 skip 로직 추가 |
| **Function UX Effect** | 검색어 하나로 SAP/SDP description, hostname, serviceId 등 모든 파싱 필드를 일관되게 검색 가능해져 결과 누락 없음 |
| **Core Value** | 파싱된 모든 필드를 빠짐없이 검색하는 완전하고 일관된 검색 경험 |

**성과 수치**

| 지표 | 결과 |
|------|------|
| Match Rate | 100% |
| 수정 버그 수 | 4건 (B1~B4) + 개선 1건 (B5) |
| 수정 파일 | 1개 (`ServiceListV3.tsx`) |
| TypeScript 컴파일 오류 | 0 |
| 반복 횟수 | 0회 |

---

## 2. 버그별 수정 내용

### B1 · B2: 단일 검색어 Epipe/VPLS SAP description·portDescription 누락

**원인**: 단일 경로에서 `sapId`만 검색하고 `description`, `portDescription` 미포함

**수정 위치**: `ServiceListV3.tsx` L663-701

```typescript
// 수정 후 (Epipe/VPLS 공통)
service.saps.some(sap =>
  sap.sapId.toLowerCase().includes(query) ||
  normalizeSearchString(sap.description).includes(query) ||       // B1
  (sap.portDescription && normalizeSearchString(sap.portDescription).includes(query)) // B2
)
```

### B3: 단일 검색어 Epipe/VPLS SDP description 누락

**수정**: Spoke SDP, Mesh SDP에 `normalizeSearchString(sdp.description)` 추가

### B3-IES: IES hostname·serviceId·description으로 검색 불가 (핵심 버그)

**원인**: IES가 `return true`로 무조건 통과 후 `filterIESInterfaces()`가 덮어씀

**수정**: 서비스 레벨 매칭을 먼저 확인하고, 매칭 시 전체 인터페이스 표시

```typescript
// 단일 경로 (L747-753)
const iesServiceLevelMatch = (
  service.serviceId.toString().includes(query) ||
  normalizeSearchString(service.description).includes(query) ||
  (service.serviceName && normalizeSearchString(service.serviceName).includes(query)) ||
  (iesHostname && normalizeSearchString(iesHostname).includes(query))
);
if (iesServiceLevelMatch) return true;

// .map() 블록 (L876-882) - 단일/복수 공통 적용
if (serviceLevelMatch) return service; // 전체 인터페이스 표시
```

### B4: 단일 경로 Catch-all Unicode 정규화 누락

**원인**: `.toLowerCase()` 사용으로 Unicode 하이픈 변형 문자 미정규화

**수정**: `normalizeSearchString(JSON.stringify(service))` 적용

### B5 (개선): VPRN portDescription 명시적 검색 추가

단일/복수 경로 모두 `iface.portDescription` 명시적 추가 (기존에는 Catch-all에서만 우연히 검색됨)

---

## 3. 영향 범위

| 서비스 타입 | 검색 개선 내용 |
|------------|--------------|
| Epipe | SAP description, portDescription, SDP description 검색 추가 |
| VPLS | SAP description, portDescription, Spoke/Mesh SDP description 검색 추가 |
| VPRN | portDescription 명시적 검색 추가 |
| IES | hostname, serviceId, description으로 서비스 레벨 검색 가능 |
| 전체 | 단일 검색어 Catch-all Unicode 하이픈 정규화 일관화 |

---

## 4. 테스트 검증

| 시나리오 | 기대 결과 | 상태 |
|----------|---------|------|
| Epipe SAP description 검색 | 해당 서비스 표시 | ✅ |
| VPLS SAP portDescription 검색 | 해당 서비스 표시 | ✅ |
| IES hostname 검색 | 전체 인터페이스 포함 표시 | ✅ |
| IES serviceId 검색 | 전체 인터페이스 포함 표시 | ✅ |
| Unicode 하이픈 hostname 검색 | 정규화 후 매칭 | ✅ |
| AND/OR 검색 회귀 없음 | 기존 동작 유지 | ✅ |

---

## 5. 관련 파일

| 파일 | 역할 |
|------|------|
| `src/components/v3/ServiceListV3.tsx` | 검색 필터 로직 수정 |
| `docs/01-plan/features/search-filter-fix.plan.md` | Plan 문서 |
| `docs/02-design/features/search-filter-fix.design.md` | Design 문서 |
