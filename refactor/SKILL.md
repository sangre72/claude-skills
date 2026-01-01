# /refactor - 모듈화 및 타입 가이드라인 검사 (user)

프로젝트의 모듈화 및 타입 가이드라인 준수 여부를 검사하고 리팩토링합니다.

## 증분 분석 원칙

**IMPORTANT**: CLAUDE.md에 소스가 추가되거나 수정된 경우, 전체 프로젝트가 아닌 **추가/변경된 부분만** 분석합니다.

- 새 파일 추가 시 → 해당 파일만 검사
- 타입 수정 시 → 해당 타입 사용처만 검사
- 모듈 추가 시 → 해당 모듈과 의존성만 검사

```bash
# 최근 변경 파일 확인
git diff --name-only HEAD~1

# 변경된 파일만 대상으로 검사
```

## 사용법

```bash
/refactor              # 전체 검사
/refactor products     # 특정 모듈만 검사
/refactor --types      # 타입 중복만 검사
/refactor --fix        # 자동 수정 포함
```

## 검사 항목

### 1. 타입 통합 (Type Consolidation)

**CRITICAL: 타입 중복 정의**
```bash
# 동일 타입이 여러 파일에 정의된 경우
grep -rln "interface Product {" apps/ packages/
grep -rln "type Product = " apps/ packages/
```

**규칙:**
- 공유 타입 → `@*/shared/types` (packages/shared/src/types/)
- 앱 전용 타입 → `apps/*/src/types/`
- 모듈 전용 타입 → `features/*/types/`

**올바른 패턴:**
```typescript
// apps/*/src/lib/api.ts
import type { Product, User } from '@auction/shared';
export type { Product, User };  // re-export for consumers
```

**잘못된 패턴:**
```typescript
// apps/*/src/lib/api.ts
export interface Product { ... }  // 중복 정의!
```

### 2. 유틸리티 통합 (Utility Consolidation)

**WARNING: 함수 중복 정의**
```bash
# 동일 함수가 여러 파일에 정의된 경우
grep -rln "function formatPrice" apps/ packages/
grep -rln "const formatPrice" apps/ packages/
```

**규칙:**
- 공유 유틸 → `@*/shared/utils`
- 앱 전용 유틸 → `apps/*/src/lib/`

### 3. 모듈 독립성

- features/* 간 직접 import 금지
- 모듈 자체 포함 구조 (components, hooks, api, types, utils)
- index.ts public API 정의

### 4. 앱 독립성 원칙 (중요)

> **CRITICAL**: 각 앱은 **권한/역할별로 서버가 완전히 분리 배포**될 수 있습니다.

```
apps/user       → user.example.com      (일반 사용자)
apps/admin      → admin.example.com     (관리자)
apps/manager    → manager.example.com   (중간 관리자, 향후)
```

**검사 원칙:**
- 앱 간 직접 import 검출 시 → **[CRITICAL]** 에러
- 앱별 동일 코드 → **허용** (중복 경고 제외)
- 공유 가능한 순수 타입/함수 중복 → **[WARNING]** 권장

**공유 대상 vs 앱 독립 유지:**

| 구분 | 공유 (`@auction/shared`) | 앱별 독립 유지 |
|------|-------------------------|---------------|
| 타입 | 백엔드 스키마 기반 (Product, User) | 앱 전용 확장 (UserProfile) |
| 유틸 | 순수 함수 (formatPrice) | 상태 관리, 앱별 로직 |
| 인증 | 팩토리 함수, 인터페이스 | 앱별 설정, 엔드포인트 |

```typescript
// ✅ 허용: 앱별 독립 (역할별 서버 분리 가능성)
// apps/user/src/lib/security.ts
// apps/admin/src/lib/security.ts

// ❌ 중복 금지: 백엔드 스키마 기반 순수 타입
// apps/user/src/types/product.ts  - interface Product
// apps/admin/src/types/product.ts - interface Product
// → @auction/shared/types로 통합 필요
```

### 5. 의존성 방향

```
@*/shared (최하위) ← @*/ui ← src/lib ← features/* ← app/* (최상위)
```

## 검사 프로세스

### Phase 1: 타입 분석
```bash
# 1. interface 중복 찾기
grep -rn "^export interface " apps/ packages/ --include="*.ts" --include="*.tsx"

# 2. type alias 중복 찾기
grep -rn "^export type " apps/ packages/ --include="*.ts" --include="*.tsx"

# 3. 백엔드 스키마와 비교 (Python 모델)
grep -rn "class.*Model" backend/ --include="*.py"
```

### Phase 2: Import 분석
```bash
# 타입 import 방식 검사
grep -rn "export type {" apps/  # 올바른 re-export
grep -rn "export interface" apps/*/src/lib/  # 잘못된 중복 정의
```

### Phase 3: 리포트
```
=== 모듈화 및 타입 검사 결과 ===

[CRITICAL] 타입 중복 정의: 2건
  - Product: apps/admin/api.ts, apps/user/api.ts
  → @auction/shared/types로 통합 필요

[WARNING] 유틸 함수 중복: 1건
  - formatDate: apps/admin/utils.ts, apps/user/utils.ts
  → @auction/shared/utils로 이동 필요

[INFO] 타입 import 방식 개선: 3건
  - export type { X } from 'module' → import type + export type

준수율: 85%
```

## 자동 수정 (--fix)

### 타입 통합 수정
1. 중복 타입을 `@auction/shared/types`로 이동
2. 원본 파일에서 import + re-export로 변경
3. 빌드 테스트

### Import 방식 수정
```typescript
// Before (파일 내에서 타입 사용 불가)
export type { Product } from '@auction/shared';

// After (파일 내에서 타입 사용 + re-export)
import type { Product } from '@auction/shared';
export type { Product };
```

## 프론트엔드 레이아웃 구조 (필수)

> **CRITICAL**: 모든 프론트엔드 프로젝트는 아래 레이아웃 구조를 **반드시** 준수해야 합니다.

### 페이지 레이아웃 구성 요소

모든 페이지는 다음 영역으로 구성됩니다:

```
┌─────────────────────────────────────────┐
│              Header                      │  ← 로고, 주요 네비게이션
├─────────────────────────────────────────┤
│           Header Utility                 │  ← 검색, 알림, 사용자 메뉴
├─────────────────────────────────────────┤
│              Menu                        │  ← 서브 네비게이션, 탭, 필터
├─────────────────────────────────────────┤
│                                          │
│             Contents                     │  ← 메인 콘텐츠 영역
│                                          │
├─────────────────────────────────────────┤
│              Footer                      │  ← 링크, 연락처, 저작권
└─────────────────────────────────────────┘
```

### 컴포넌트 구조

```
components/
├── common/                    # 공통 레이아웃 컴포넌트
│   ├── Header.tsx            # 헤더 (로고, 메인 네비게이션)
│   ├── HeaderUtility.tsx     # 헤더 유틸리티 (검색, 알림, 사용자)
│   ├── Menu.tsx              # 메뉴/네비게이션
│   ├── Footer.tsx            # 푸터
│   ├── Layout.tsx            # 전체 레이아웃 래퍼
│   └── MobileMenu.tsx        # 모바일 메뉴 (햄버거)
├── auth/                      # 인증 관련 컴포넌트
├── forms/                     # 폼 컴포넌트
└── [domain]/                  # 도메인별 컴포넌트
```

### 반응형 처리 (필수)

**모든 컴포넌트는 반응형 디자인을 적용해야 합니다:**

| 브레이크포인트 | MUI | 설명 |
|---------------|-----|------|
| Mobile | `xs` (0-599px) | 햄버거 메뉴, 1열 레이아웃 |
| Tablet | `sm` (600-899px) | 축소된 네비게이션, 2열 가능 |
| Desktop | `md` (900-1199px) | 풀 네비게이션, 다중 열 |
| Large | `lg` (1200px+) | 최대 너비 제한 (maxWidth) |

**반응형 패턴:**
```typescript
// MUI sx 속성 사용
<Box sx={{
  display: { xs: 'none', md: 'flex' },      // 모바일 숨김
  flexDirection: { xs: 'column', md: 'row' },
  gap: { xs: 1, sm: 2, md: 3 },
}}>

// Header 예시
<AppBar>
  {/* 데스크톱: 풀 메뉴 */}
  <Box sx={{ display: { xs: 'none', md: 'flex' } }}>
    <NavLinks />
  </Box>

  {/* 모바일: 햄버거 메뉴 */}
  <IconButton sx={{ display: { xs: 'flex', md: 'none' } }}>
    <MenuIcon />
  </IconButton>
</AppBar>
```

### 레이아웃 검사 항목

```bash
# Header/Footer 존재 확인
grep -rln "Header" app/ pages/ --include="*.tsx"
grep -rln "Footer" app/ pages/ --include="*.tsx"

# 반응형 처리 확인
grep -rln "display: { xs:" components/ --include="*.tsx"
grep -rln "useMediaQuery" components/ --include="*.tsx"
```

**체크리스트:**
- [ ] Header 컴포넌트 분리 (common/Header.tsx)
- [ ] Footer 컴포넌트 분리 (common/Footer.tsx)
- [ ] 모바일 메뉴 구현 (햄버거/드로어)
- [ ] 모든 페이지에 일관된 레이아웃 적용
- [ ] 브레이크포인트별 UI 테스트

---

## 프론트엔드 모듈화 (필수)

> **CRITICAL**: 각 영역에서 사용되는 컴포넌트, 훅, 유틸리티는 **철저하게 모듈화**해야 합니다.

### 모듈화 원칙

**1. 영역별 분리**
```
src/
├── components/
│   ├── common/      # 레이아웃, 공통 UI (Header, Footer, Button)
│   ├── auth/        # 인증 (LoginForm, RegisterForm)
│   ├── forms/       # 폼 요소 (Input, Select, DatePicker)
│   └── [domain]/    # 도메인별 (ProductCard, UserProfile)
├── hooks/
│   ├── useAuth.ts        # 인증 상태
│   ├── useApi.ts         # API 호출
│   ├── useMediaQuery.ts  # 반응형
│   └── use[Domain].ts    # 도메인별 훅
├── lib/
│   ├── api.ts       # API 클라이언트
│   ├── utils.ts     # 유틸리티 함수
│   └── constants.ts # 상수 정의
├── stores/
│   └── [name]Store.ts    # 전역 상태
└── types/
    └── index.ts     # 타입 정의
```

**2. 단일 책임 원칙**
```typescript
// ❌ 잘못된 예: 하나의 컴포넌트에 모든 로직
function Header() {
  const [user, setUser] = useState();
  const [notifications, setNotifications] = useState();
  const [searchQuery, setSearchQuery] = useState();
  // 100줄 이상의 로직...
}

// ✅ 올바른 예: 역할별 분리
// components/common/Header.tsx
function Header() {
  return (
    <AppBar>
      <Logo />
      <Navigation />
      <HeaderUtility />
    </AppBar>
  );
}

// components/common/HeaderUtility.tsx
function HeaderUtility() {
  const { user } = useAuth();
  const { notifications } = useNotifications();
  return (<SearchBar /><NotificationBell /><UserMenu />);
}
```

**3. index.ts를 통한 Public API**
```typescript
// hooks/index.ts
export { useAuth } from './useAuth';
export { useApi } from './useApi';
export { useMediaQuery } from './useMediaQuery';

// 사용처
import { useAuth, useApi } from '@/hooks';
```

**4. 컴포넌트 크기 제한**
- 단일 컴포넌트: 최대 200줄
- 200줄 초과 시: 하위 컴포넌트로 분리
- 복잡한 로직: 커스텀 훅으로 추출

### 모듈화 검사 항목

```bash
# 컴포넌트 크기 검사 (200줄 초과)
find components/ -name "*.tsx" -exec wc -l {} \; | awk '$1 > 200'

# index.ts 존재 확인
ls -la components/index.ts hooks/index.ts lib/index.ts

# 공통 컴포넌트 분리 확인
ls -la components/common/
```

**체크리스트:**
- [ ] 레이아웃 컴포넌트 분리 (Header, Footer, Menu)
- [ ] 도메인별 컴포넌트 폴더 구성
- [ ] 커스텀 훅 분리 (hooks/)
- [ ] 유틸리티 함수 분리 (lib/)
- [ ] 타입 중앙 관리 (types/)
- [ ] index.ts를 통한 export
- [ ] 200줄 이상 컴포넌트 분리

---

## 백엔드 스키마 동기화

Python FastAPI 모델과 TypeScript 타입 일치 여부 검사:

```bash
# 백엔드 모델 필드
grep -A 20 "class Product" backend/products/models.py

# 프론트엔드 타입 필드
grep -A 20 "interface Product" packages/shared/src/types/
```

**체크리스트:**
- [ ] 필드명 일치 (snake_case)
- [ ] 타입 일치 (number/string/boolean)
- [ ] Optional 필드 일치 (?)
- [ ] Enum 값 일치

## 출력 예시

```
[CRITICAL] 타입 중복 정의
  apps/admin/src/lib/api.ts:45 - interface Product
  apps/user/src/lib/api.ts:32 - interface Product
  packages/shared/src/types/index.ts:45 - interface Product
  → packages/shared로 통합, 나머지는 import로 변경

[WARNING] 유틸 함수 중복
  apps/admin/src/components/ProductCard.tsx:8 - formatPrice
  apps/user/src/components/AuctionCard.tsx:12 - formatPrice
  → @auction/shared/utils로 이동 완료됨 ✓

[INFO] Import 방식 개선 필요
  apps/admin/src/lib/api.ts:5
  export type { Product } from '@auction/shared'
  → import type { Product } from '@auction/shared'; export type { Product };
```
