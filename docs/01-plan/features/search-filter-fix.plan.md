# Plan: 검색 필터 버그 수정 (search-filter-fix)

> **작성일**: 2026-06-02
> **버전**: v5.9.0 예정
> **상태**: 🟡 Plan 단계

---

## Executive Summary

| 관점 | 내용 |
|------|------|
| **Problem** | 검색창 단일 검색어에서 Epipe/VPLS SAP·SDP description이 누락되고, IES 서비스는 hostname·serviceId·description으로 검색 불가하며, VPRN portDescription도 Catch-all에서만 우연히 검색됨 |
| **Solution** | 단일 검색어 로직을 복수 검색어(AND/OR) 로직과 대칭적으로 수정하고, IES 필터링 흐름을 재구조화하여 서비스 레벨 매칭과 인터페이스 레벨 필터링을 분리 |
| **Function UX Effect** | 검색어를 입력했을 때 결과가 일관되게 나오며, IES 서비스도 장비명·서비스ID로 검색 가능해져 사용자가 검색 결과를 신뢰할 수 있음 |
| **Core Value** | 파싱된 모든 필드를 빠짐없이 검색하는 완전하고 일관된 검색 경험 제공 |

---

## 1. 배경 및 목적

### 1.1 문제 상황

`ServiceListV3.tsx`의 검색 로직은 단일 검색어(공백 없음)와 복수 검색어(AND/OR) 두 경로로 분기된다. 이 두 경로가 대칭적으로 구현되지 않아 아래 4가지 버그가 존재한다.

### 1.2 발견된 버그

| # | 버그 | 영향 서비스 타입 | 심각도 |
|---|------|----------------|--------|
| B1 | 단일 검색어에서 SAP/SDP `description` 미검색 | Epipe, VPLS | 높음 |
| B2 | 단일 검색어에서 SAP `portDescription` 미검색 | Epipe, VPLS | 높음 |
| B3 | IES를 hostname/serviceId/description으로 검색 불가 | IES | 높음 |
| B4 | 단일 검색어 Catch-all에서 `normalizeSearchString` 미적용 (`.toLowerCase()` 사용) | 전체 | 중간 |

> **B5 (낮음)**: VPRN `portDescription`은 명시적 검색 없이 Catch-all에서만 검색됨. 현재 동작 자체는 되므로 이번에 명시적 추가.

### 1.3 근본 원인

- 단일 검색어 최적화 경로(`searchTerms.length === 1`)가 복수 경로와 별개로 작성되어 필드 범위가 다름
- IES의 `return true` 패턴이 서비스 레벨 검색(hostname, serviceId 등)을 무력화함
- `filterIESInterfaces()`가 서비스 레벨 매칭 여부와 관계없이 항상 인터페이스 필터링을 덮어씀

---

## 2. 범위 (Scope)

### 2.1 수정 대상 파일

- `src/components/v3/ServiceListV3.tsx` — 검색 필터 로직 전체

### 2.2 수정 범위

**포함:**
- B1: 단일 검색어 Epipe/VPLS SAP `description`, `portDescription` 추가
- B2: 단일 검색어 Epipe/VPLS SDP `description` 추가
- B3: IES 서비스 레벨 매칭(hostname, serviceId, description) → 매칭 시 인터페이스 필터 없이 전체 표시
- B4: 단일 검색어 Catch-all `normalizeSearchString` 적용
- B5: VPRN `portDescription` 명시적 검색 추가 (단일/복수 모두)

**제외:**
- 검색 UI 변경 없음
- 파서(parserV3.ts) 변경 없음
- 파일시스템 검색(search-global-config) 변경 없음
- IES 인터페이스 레벨 필터링 동작 자체는 유지 (서비스 레벨 미매칭 시에만 인터페이스 필터 적용)

---

## 3. 요구사항

### 3.1 기능 요구사항

| ID | 요구사항 | 우선순위 |
|----|----------|--------|
| FR-01 | 단일 검색어로 Epipe/VPLS SAP description 검색 가능 | 필수 |
| FR-02 | 단일 검색어로 Epipe/VPLS SAP portDescription 검색 가능 | 필수 |
| FR-03 | 단일 검색어로 Epipe/VPLS SDP description 검색 가능 | 필수 |
| FR-04 | IES 서비스를 hostname으로 검색 시 전체 인터페이스 표시 | 필수 |
| FR-05 | IES 서비스를 serviceId로 검색 시 전체 인터페이스 표시 | 필수 |
| FR-06 | IES 서비스를 description으로 검색 시 전체 인터페이스 표시 | 필수 |
| FR-07 | 단일 검색어 Catch-all에서 Unicode 하이픈 정규화 적용 | 필수 |
| FR-08 | VPRN 인터페이스 portDescription 명시적 검색 (단일/복수) | 권장 |

### 3.2 비기능 요구사항

| ID | 요구사항 |
|----|----------|
| NFR-01 | 단일/복수 검색어 경로 결과가 동일한 검색어에서 일치해야 함 |
| NFR-02 | 기존 정상 동작하던 검색 결과에 회귀 없음 |
| NFR-03 | TypeScript strict mode 컴파일 오류 없음 |

---

## 4. 성공 기준

| 기준 | 목표 |
|------|------|
| 설계-구현 Match Rate | ≥ 90% |
| TypeScript 컴파일 에러 | 0 |
| 기존 검색 동작 회귀 | 없음 |
| IES hostname 검색 동작 | ✅ |
| 단일/복수 검색어 결과 일관성 | ✅ |
