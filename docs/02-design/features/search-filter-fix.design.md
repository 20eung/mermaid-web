# Design: 검색 필터 버그 수정 (search-filter-fix)

> **작성일**: 2026-06-02
> **Plan 참조**: `docs/01-plan/features/search-filter-fix.plan.md`
> **수정 대상**: `src/components/v3/ServiceListV3.tsx`

---

## 1. 문제 분석

### 1.1 현재 코드 구조

```
searchQuery 입력
  ├─ IP 주소 판별 → IP 서브넷 LPM 검색 (별도 경로, 수정 불필요)
  └─ 문자열 검색
       ├─ searchTerms.length === 1  ← [단일 경로] 버그 존재
       └─ searchTerms.length > 1   ← [복수 경로] 정상 (AND/OR)
```

**단일 경로의 문제점:**
1. Epipe/VPLS: `sapId`만 검색, `description`·`portDescription` 누락
2. IES: `return true` 후 `filterIESInterfaces()`가 덮어써서 서비스 레벨 매칭 불가
3. Catch-all: `.toLowerCase()` 사용 (normalizeSearchString 미적용)

### 1.2 IES 필터링 현재 흐름 (버그)

```
service.serviceType === 'ies'
  → return true  (무조건 통과)
  → filterIESInterfaces() 적용  ← 인터페이스 미매칭 시 null 반환
  → hostname/serviceId/description으로 검색해도 제외됨
```

---

## 2. 수정 설계

### 2.1 단일 검색어 Epipe/VPLS 수정 (FR-01~03)

**현재** ([ServiceListV3.tsx:662-688](../../src/components/v3/ServiceListV3.tsx)):
```typescript
if (service.serviceType === 'epipe') {
  if ('saps' in service && service.saps) {
    if (service.saps.some(sap => sap.sapId.toLowerCase().includes(query))) return true;
  }
  if ('spokeSdps' in service && service.spokeSdps) {
    if (service.spokeSdps.some(sdp =>
      sdp.sdpId.toString().includes(query) ||
      sdp.vcId.toString().includes(query)
    )) return true;
  }
}
```

**변경 후:**
```typescript
if (service.serviceType === 'epipe') {
  if ('saps' in service && service.saps) {
    if (service.saps.some(sap =>
      sap.sapId.toLowerCase().includes(query) ||
      normalizeSearchString(sap.description).includes(query) ||    // FR-01
      (sap.portDescription && normalizeSearchString(sap.portDescription).includes(query)) // FR-02
    )) return true;
  }
  if ('spokeSdps' in service && service.spokeSdps) {
    if (service.spokeSdps.some(sdp =>
      sdp.sdpId.toString().includes(query) ||
      sdp.vcId.toString().includes(query) ||
      normalizeSearchString(sdp.description).includes(query)       // FR-03
    )) return true;
  }
}
// VPLS도 동일하게 적용 (saps + spokeSdps + meshSdps description 추가)
```

### 2.2 VPRN portDescription 명시적 추가 (FR-08)

단일 경로 [ServiceListV3.tsx:693-706](../../src/components/v3/ServiceListV3.tsx):
```typescript
// 추가
if (iface.portDescription && normalizeSearchString(iface.portDescription).includes(query)) return true;
```

복수 경로 [ServiceListV3.tsx:789-798](../../src/components/v3/ServiceListV3.tsx):
```typescript
searchFields.push(
  iface.interfaceName || '',
  iface.description || '',
  iface.portId || '',
  iface.ipAddress || '',
  iface.vplsName || '',
  iface.spokeSdpId || '',
  iface.portDescription || '',  // FR-08 추가
);
```

### 2.3 IES 서비스 레벨 매칭 재구조화 (FR-04~06)

**변경 후 흐름:**
```
service.serviceType === 'ies'
  → 서비스 레벨 매칭 먼저 확인
      ├─ serviceId, description, hostname 매칭 시
      │    → return true (인터페이스 필터 없이 전체 표시)
      └─ 미매칭 시
           → filterIESInterfaces() 적용 (인터페이스 레벨 필터)
```

**단일 경로 IES 변경 ([ServiceListV3.tsx:736-739](../../src/components/v3/ServiceListV3.tsx)):**
```typescript
} else if (service.serviceType === 'ies') {
  // 서비스 레벨 매칭 먼저 확인 (hostname, serviceId, description)
  const iesHostname = (service as any)._hostname;
  const serviceLevelMatch = (
    service.serviceId.toString().includes(query) ||
    normalizeSearchString(service.description).includes(query) ||
    (service.serviceName && normalizeSearchString(service.serviceName).includes(query)) ||
    (iesHostname && normalizeSearchString(iesHostname).includes(query))
  );
  if (serviceLevelMatch) return true;
  // 서비스 레벨 미매칭 → 아래 Catch-all(JSON) 또는 인터페이스 필터로 처리
}
```

**IES 인터페이스 필터 적용 조건 변경 ([ServiceListV3.tsx:856-864](../../src/components/v3/ServiceListV3.tsx)):**
```typescript
// 현재: IES는 무조건 filterIESInterfaces 적용
// 변경: 서비스 레벨 JSON 매칭이 된 경우 필터 건너뜀

.map(service => {
  if (service.serviceType === 'ies' && searchQuery) {
    const normalizedQuery = normalizeSearchString(searchQuery);
    // 서비스 레벨 필드 매칭 재확인
    const iesHostname = (service as any)._hostname;
    const serviceLevelMatch = (
      service.serviceId.toString().includes(normalizedQuery) ||
      normalizeSearchString(service.description).includes(normalizedQuery) ||
      (service.serviceName && normalizeSearchString(service.serviceName).includes(normalizedQuery)) ||
      (iesHostname && normalizeSearchString(iesHostname).includes(normalizedQuery))
    );
    if (serviceLevelMatch) return service; // 전체 인터페이스 표시
    return filterIESInterfaces(
      service as IESService & { _hostname: string },
      normalizedQuery
    );
  }
  return service;
})
```

> **AND/OR 복수 경로 IES**: 현재 `return true`로 통과 후 동일한 `.map()` 블록에서 `filterIESInterfaces` 적용. 위 `.map()` 수정으로 복수 경로도 동시에 수정됨.

### 2.4 단일 경로 Catch-all 정규화 수정 (B4)

**현재 ([ServiceListV3.tsx:746](../../src/components/v3/ServiceListV3.tsx)):**
```typescript
const serviceJson = JSON.stringify(service).toLowerCase();
```

**변경 후:**
```typescript
const serviceJson = normalizeSearchString(JSON.stringify(service));
```

---

## 3. 수정 파일 및 위치 요약

| # | 파일 | 위치 | 변경 내용 |
|---|------|------|----------|
| 1 | `ServiceListV3.tsx` | L664-688 (단일, epipe) | SAP description/portDescription, SDP description 추가 |
| 2 | `ServiceListV3.tsx` | L674-689 (단일, vpls) | SAP description/portDescription, SDP/MeshSDP description 추가 |
| 3 | `ServiceListV3.tsx` | L700 (단일, vprn) | iface.portDescription 추가 |
| 4 | `ServiceListV3.tsx` | L736-739 (단일, ies) | 서비스 레벨 매칭 로직으로 교체 |
| 5 | `ServiceListV3.tsx` | L746 (단일, catch-all) | `.toLowerCase()` → `normalizeSearchString()` |
| 6 | `ServiceListV3.tsx` | L795 (복수, vprn) | iface.portDescription 추가 |
| 7 | `ServiceListV3.tsx` | L856-864 (ies map) | 서비스 레벨 매칭 시 필터 skip 로직 추가 |

---

## 4. 테스트 시나리오

| 시나리오 | 검색어 예시 | 기대 결과 |
|----------|-----------|---------|
| Epipe SAP description 검색 | `"customer-a"` (SAP description에 있는 값) | 해당 Epipe 서비스 표시 |
| Epipe SAP portDescription 검색 | `"uplink"` (port description에 있는 값) | 해당 Epipe 서비스 표시 |
| IES hostname 검색 | `"router-01"` | IES 서비스 전체 인터페이스 포함 표시 |
| IES serviceId 검색 | `"1001"` | IES 서비스 전체 인터페이스 포함 표시 |
| Unicode 하이픈 포함 hostname | `"router‑01"` (U+2011) | 정규화 후 매칭 |
| VPRN portDescription 검색 | `"wan-port"` | VPRN 서비스 표시 |

---

## 5. 구현 순서

1. 단일 경로 Epipe/VPLS SAP·SDP description 추가 (B1, B2, B3)
2. 단일 경로 VPRN portDescription 추가 (B5)
3. 단일 경로 IES 서비스 레벨 매칭 교체 (B3-IES)
4. 단일 경로 Catch-all normalizeSearchString 적용 (B4)
5. 복수 경로 VPRN portDescription 추가 (B5)
6. IES `.map()` 서비스 레벨 skip 로직 추가 (B3-IES)
7. TypeScript 컴파일 확인
