# 코딩 가이드라인 생성 스킬

사용자가 `/coding-guide` 명령어를 실행하면 프로젝트에 맞는 코딩 가이드라인을 생성하고 CLAUDE.md에 추가합니다.

## 증분 분석 원칙

**IMPORTANT**: CLAUDE.md에 소스가 추가되거나 수정된 경우, 전체 프로젝트가 아닌 **추가/변경된 부분만** 분석합니다.

- 새 언어/프레임워크 추가 시 → 해당 언어 규칙만 추가
- 기존 규칙 수정 시 → 변경된 규칙만 업데이트
- 새 파일 패턴 추가 시 → 해당 패턴 규칙만 추가

```bash
# 최근 변경 파일 확인
git diff --name-only HEAD~1

# 새로 추가된 파일 타입 확인
git diff --name-only HEAD~1 | xargs -I{} basename {} | sed 's/.*\.//' | sort -u
```

## 실행 조건

- 사용자가 `/coding-guide` 실행
- "코딩 가이드 만들어줘" 요청

## 실행 단계

### 1단계: 프로젝트 분석

```bash
# 프로젝트 타입 확인
ls package.json tsconfig.json pyproject.toml requirements.txt go.mod Cargo.toml 2>/dev/null

# 기존 CLAUDE.md 확인
cat CLAUDE.md 2>/dev/null || echo "없음"

# 기존 가이드라인 파일 확인
ls CODING_GUIDELINES.md CONTRIBUTING.md .editorconfig 2>/dev/null
```

### 2단계: 언어별 가이드라인 생성

#### TypeScript/JavaScript
```markdown
## 코딩 규칙

### TypeScript
- `any` 사용 금지 → `unknown` 사용
- `strict: true` 모드 준수
- 명시적 타입 선언 필수

### 네이밍 컨벤션
- 변수/함수: `camelCase` (예: `getUserInfo`)
- 상수: `UPPER_SNAKE_CASE` (예: `MAX_RETRIES`)
- 컴포넌트/클래스: `PascalCase` (예: `UserProfile`)
- Boolean: `is`, `has`, `should` 접두사 (예: `isLoading`)
- 이벤트 핸들러: `handle` 접두사 (예: `handleClick`)

### 파일명
- 컴포넌트: `PascalCase.tsx` (예: `UserCard.tsx`)
- 유틸리티: `camelCase.ts` (예: `formatDate.ts`)
- 타입: `types.ts` 또는 `*.types.ts`
- 테스트: `*.test.ts`, `*.spec.ts`

### Import 순서
1. 외부 라이브러리 (react, next 등)
2. 내부 패키지 (@auction/shared 등)
3. 상대 경로 (../components 등)
4. 타입 import (type { ... })

### 에러 처리
- try-catch 블록에서 구체적인 에러 타입 처리
- 사용자에게 보여줄 메시지와 로깅 메시지 분리
- 민감 정보 로깅 금지
```

#### Python
```markdown
## 코딩 규칙

### Python
- Type hints 필수 사용
- PEP 8 스타일 가이드 준수
- docstring 필수 (Google 스타일)

### 네이밍 컨벤션
- 변수/함수: `snake_case` (예: `get_user_info`)
- 상수: `UPPER_SNAKE_CASE` (예: `MAX_RETRIES`)
- 클래스: `PascalCase` (예: `UserService`)
- Private: `_` 접두사 (예: `_internal_method`)

### 파일명
- 모듈: `snake_case.py` (예: `user_service.py`)
- 테스트: `test_*.py` (예: `test_user_service.py`)

### Import 순서
1. 표준 라이브러리
2. 서드파티 라이브러리
3. 로컬 모듈

### 에러 처리
- 커스텀 Exception 클래스 사용
- 구체적인 예외 타입 catch
- 로깅 시 민감정보 마스킹
```

### 3단계: CLAUDE.md 업데이트

기존 CLAUDE.md가 있으면 코딩 규칙 섹션 추가/업데이트, 없으면 새로 생성:

```markdown
# Claude Code 프로젝트 가이드

## 프로젝트 개요
- **프로젝트명**: {프로젝트명}
- **기술 스택**: {감지된 기술 스택}

## 코딩 규칙

{언어별 가이드라인}

## 보안 라이브러리 규칙 (필수)

> **CRITICAL**: 보안 관련 기능은 반드시 **취약점이 없고 공격에 안전한** 검증된 라이브러리와 방식만 사용합니다.
>
> 코드 생성 시 OWASP Top 10 취약점(Injection, XSS, CSRF 등)을 반드시 방지해야 합니다.

### Python (FastAPI)

| 기능 | 필수 라이브러리 | 금지 |
|------|----------------|------|
| JWT | `python-jose[cryptography]` | `PyJWT` 단독 사용 |
| 비밀번호 해싱 | `passlib[bcrypt]`, `bcrypt` | 직접 구현, MD5, SHA1 |
| 암호화 | `cryptography` | 직접 구현 |
| HTTP 클라이언트 | `httpx` (async) | `urllib` 직접 사용 |

```python
# 올바른 import
from jose import jwt  # python-jose 사용
import bcrypt         # passlib과 함께 사용

# 금지
import jwt  # PyJWT 단독 - python-jose와 충돌
```

### TypeScript/JavaScript

| 기능 | 필수 라이브러리 | 금지 |
|------|----------------|------|
| JWT | `jose` | `jsonwebtoken` 단독 |
| 암호화 | `crypto` (Node.js 내장) | 직접 구현 |
| 인증 쿠키 | `httpOnly`, `secure`, `sameSite` 필수 | 일반 쿠키에 토큰 저장 |

### 인증 쿠키 필수 설정

```python
response.set_cookie(
    key="token",
    value=token,
    httponly=True,      # JavaScript 접근 차단
    secure=True,        # HTTPS 전용 (프로덕션)
    samesite="lax",     # CSRF 방지
)
```

### 취약점 방지 필수 사항

| 취약점 | 방지 방법 |
|--------|----------|
| SQL Injection | ORM 사용, 파라미터 바인딩 필수 |
| XSS | 입력값 이스케이프, CSP 헤더 |
| CSRF | SameSite 쿠키, CSRF 토큰 |
| 인증 우회 | JWT 검증, 세션 관리 |
| 민감정보 노출 | 환경변수, 로그 마스킹 |

### 금지 사항

- 비밀번호 평문 저장/로깅
- 토큰을 localStorage에 저장 (XSS 취약)
- 직접 암호화 알고리즘 구현 (취약점 위험)
- 환경변수 없이 시크릿 키 하드코딩
- SQL 문자열 직접 조합 (Injection 취약)
- 사용자 입력값 검증 없이 사용

### JWT 인증 필수

> **중요**: 공개 API를 제외한 모든 API는 JWT access token 검증이 필수입니다.

| 구분 | 인증 필요 | 예시 |
|------|----------|------|
| 공개 API | ❌ | 로그인, 회원가입, 공개 상품 조회 |
| 일반 API | ✅ | 상품 CRUD, 사용자 정보, 주문 |
| 관리자 API | ✅ + 권한 | 사용자 관리, 시스템 설정 |

```python
# 백엔드 - 인증 필수 API (기본)
@router.get("/products")
async def list_products(
    current_user: User = Depends(get_current_user),  # 필수
    db: Session = Depends(get_db)
):
    pass

# 공개 API는 예외적으로 명시
@router.post("/auth/login")  # 공개
@router.get("/products/public/{id}")  # 공개 (명시적 prefix)
```

```typescript
// 프론트엔드 - API 호출 시 항상 토큰 포함
const response = await fetch('/api/products', {
  headers: { 'Authorization': `Bearer ${accessToken}` },
  credentials: 'include',
});
```

### 프로덕션 에러 응답 규칙

> **중요**: 프로덕션에서는 HTTP 상태 코드(404, 500 등)를 사용자에게 직접 노출하지 않습니다.

| 항목 | 개발 환경 | 프로덕션 환경 |
|------|----------|-------------|
| HTTP 상태 코드 | 실제 코드 (404, 500) | 항상 200 |
| 에러 메시지 | 상세 메시지 | 일반 메시지 |
| 스택 트레이스 | 포함 | ❌ 절대 금지 |
| 성공/실패 판단 | `success` 필드 | `success` 필드 |

```python
# 백엔드 - 프로덕션 에러 응답
@app.exception_handler(Exception)
async def global_exception_handler(request: Request, exc: Exception):
    return JSONResponse(
        status_code=200,  # 항상 200 반환
        content={
            "success": False,
            "message": "요청을 처리할 수 없습니다.",
            "error_code": "INTERNAL_ERROR"
        }
    )
```

```typescript
// 프론트엔드 - success 필드로 성공/실패 판단
const data = await response.json();
if (!data.success) {
  throw new ApiError(data.message, data.error_code);
}

// ❌ 잘못된 예 - HTTP 상태 코드로 판단
if (response.status === 404) { }  // 프로덕션에서는 항상 200
```

**에러 코드 매핑**:
| 내부 상태 | error_code | 사용자 메시지 |
|----------|------------|--------------|
| 401 | `AUTH_REQUIRED` | 로그인이 필요합니다 |
| 403 | `ACCESS_DENIED` | 접근 권한이 없습니다 |
| 404 | `NOT_FOUND` | 요청한 정보를 찾을 수 없습니다 |
| 500 | `INTERNAL_ERROR` | 요청을 처리할 수 없습니다 |

## 데이터베이스 테이블 설계 규칙 (필수)

> **CRITICAL**: 모든 데이터베이스 테이블은 아래 필수 컬럼을 반드시 포함해야 합니다.

### 필수 컬럼 (모든 테이블)

| 컬럼명 | 타입 | 설명 | 기본값 |
|--------|------|------|--------|
| `id` | UUID / BIGINT | Primary Key | auto-generated |
| `created_at` | TIMESTAMP | 생성일시 | `NOW()` / `CURRENT_TIMESTAMP` |
| `created_by` | UUID / VARCHAR | 생성자 ID | NULL (시스템 생성 허용) |
| `updated_at` | TIMESTAMP | 수정일시 | `NOW()` / `CURRENT_TIMESTAMP` |
| `updated_by` | UUID / VARCHAR | 수정자 ID | NULL |
| `is_active` | BOOLEAN | 사용 여부 | `TRUE` |
| `is_deleted` | BOOLEAN | 삭제 여부 (Soft Delete) | `FALSE` |

### 데이터베이스별 날짜 함수

| 데이터베이스 | 현재 시간 함수 | 사용 예시 |
|-------------|---------------|----------|
| PostgreSQL | `NOW()` / `CURRENT_TIMESTAMP` | `DEFAULT NOW()` |
| MySQL | `NOW()` / `CURRENT_TIMESTAMP` | `DEFAULT CURRENT_TIMESTAMP` |
| Oracle | `SYSDATE` / `SYSTIMESTAMP` | `DEFAULT SYSDATE` |
| SQLite | `CURRENT_TIMESTAMP` | `DEFAULT CURRENT_TIMESTAMP` |
| SQL Server | `GETDATE()` / `SYSDATETIME()` | `DEFAULT GETDATE()` |

### SQLAlchemy 모델 예시 (Python)

```python
from datetime import datetime
from typing import Optional
import uuid

from sqlalchemy import Boolean, ForeignKey, String
from sqlalchemy.dialects.postgresql import UUID
from sqlalchemy.orm import Mapped, mapped_column
from sqlalchemy.sql import func

class TimestampMixin:
    """모든 모델에 필수로 포함되는 Mixin."""

    created_at: Mapped[datetime] = mapped_column(
        default=func.now(),
        nullable=False,
    )
    created_by: Mapped[Optional[uuid.UUID]] = mapped_column(
        UUID(as_uuid=True),
        nullable=True,
    )
    updated_at: Mapped[datetime] = mapped_column(
        default=func.now(),
        onupdate=func.now(),
        nullable=False,
    )
    updated_by: Mapped[Optional[uuid.UUID]] = mapped_column(
        UUID(as_uuid=True),
        nullable=True,
    )
    is_active: Mapped[bool] = mapped_column(Boolean, default=True)
    is_deleted: Mapped[bool] = mapped_column(Boolean, default=False)


class User(Base, TimestampMixin):
    """사용자 모델 - TimestampMixin 상속 필수."""

    __tablename__ = "users"

    id: Mapped[uuid.UUID] = mapped_column(
        UUID(as_uuid=True),
        primary_key=True,
        default=uuid.uuid4,
    )
    # ... 기타 필드
```

### Prisma 스키마 예시 (TypeScript)

```prisma
model User {
  id         String   @id @default(uuid())

  // ... 기타 필드

  // 필수 컬럼
  createdAt  DateTime @default(now()) @map("created_at")
  createdBy  String?  @map("created_by")
  updatedAt  DateTime @updatedAt @map("updated_at")
  updatedBy  String?  @map("updated_by")
  isActive   Boolean  @default(true) @map("is_active")
  isDeleted  Boolean  @default(false) @map("is_deleted")

  @@map("users")
}
```

### 프론트엔드 타입 예시 (TypeScript)

```typescript
// 모든 엔티티가 상속해야 하는 기본 타입
interface BaseEntity {
  id: string;
  createdAt: string;  // ISO 8601 형식
  createdBy?: string;
  updatedAt: string;
  updatedBy?: string;
  isActive: boolean;
  isDeleted: boolean;
}

interface User extends BaseEntity {
  name: string;
  email: string;
  // ... 기타 필드
}
```

### 쿼리 시 주의사항

```python
# ✅ 올바른 예: 삭제되지 않은 활성 데이터만 조회
query = select(User).where(
    User.is_deleted == False,
    User.is_active == True,
)

# ❌ 잘못된 예: 삭제/비활성 필터 없음
query = select(User)  # Soft Delete된 데이터도 조회됨
```

### 체크리스트

- [ ] 모든 테이블에 `created_at`, `updated_at` 포함
- [ ] 모든 테이블에 `created_by`, `updated_by` 포함
- [ ] 모든 테이블에 `is_active`, `is_deleted` 포함
- [ ] 날짜 기본값에 DB에 맞는 함수 사용 (`NOW()`, `SYSDATE` 등)
- [ ] `updated_at`에 자동 업데이트 트리거/옵션 설정
- [ ] 조회 쿼리에 `is_deleted = FALSE` 필터 적용
- [ ] 백엔드 모델과 프론트엔드 타입 동기화

---

## 프론트엔드 UX 규칙 (필수)

> **CRITICAL**: 사용자 경험을 위해 모든 폼과 다단계 프로세스에서 아래 규칙을 준수합니다.

### 네비게이션 필수 요소

| 상황 | 필수 요소 | 설명 |
|------|----------|------|
| 다단계 폼/프로세스 | "이전으로" 버튼 | 이전 단계로 돌아가기 |
| 가입/신청 프로세스 | "취소" 링크 | 프로세스 중단 및 이탈 |
| 모달/팝업 | 닫기 버튼 (X) | 모달 닫기 |
| 상세 페이지 | "목록으로" 버튼 | 목록 페이지로 복귀 |

### 다단계 폼 필수 패턴

```tsx
// ✅ 올바른 예: 모든 단계에 이전/취소 버튼 포함
const renderStep = () => (
  <Box>
    {/* 메인 콘텐츠 */}

    {/* 다음 단계 버튼 */}
    <Button variant="contained" onClick={handleNext}>
      다음
    </Button>

    {/* 이전 단계 버튼 (첫 단계 제외) */}
    {step !== 'first' && (
      <Button variant="text" onClick={handlePrevious}>
        이전으로
      </Button>
    )}
  </Box>
);

// 페이지 하단에 취소 링크
{step !== 'first' && (
  <Typography onClick={() => router.push('/')}>
    취소
  </Typography>
)}
```

### 체크리스트

- [ ] 다단계 폼의 모든 단계에 "이전으로" 버튼 추가 (첫 단계 제외)
- [ ] 가입/신청 프로세스에 "취소" 링크 추가
- [ ] 모달에 닫기 버튼 (X) 또는 바깥 영역 클릭 닫기 구현
- [ ] 상세 페이지에 "목록으로" 또는 뒤로가기 버튼 추가
- [ ] 폼 제출 실패 시 입력값 유지
- [ ] 로딩 상태에서 버튼 비활성화

### 에러 복구

```tsx
// ✅ 에러 발생 시 사용자가 복구할 수 있는 옵션 제공
{error && (
  <Alert severity="error">
    {error}
    <Button onClick={handleRetry}>다시 시도</Button>
  </Alert>
)}
```

---

## 일반 보안 규칙
- 환경 변수 검증 필수
- SQL Injection, XSS 방지
- 민감 정보 하드코딩 금지
- `.env` 파일 절대 커밋 금지

## Git 커밋 규칙
Conventional Commits 형식:
- `feat`: 새로운 기능
- `fix`: 버그 수정
- `docs`: 문서 변경
- `refactor`: 리팩토링
- `test`: 테스트
- `chore`: 빌드, 설정

## 사용 가능한 스킬
- `/gitpush` - 자동 커밋 및 push
- `/gitpull` - dev 브랜치 merge 후 pull
```

### 4단계: 선택적 - 별도 파일 생성

사용자 요청 시 `CODING_GUIDELINES.md` 파일도 생성:

```bash
# 상세 가이드라인 파일 생성
# CLAUDE.md는 요약, CODING_GUIDELINES.md는 상세
```

## 완료 메시지

```
✅ 코딩 가이드라인 생성 완료!

업데이트된 파일:
- CLAUDE.md (코딩 규칙 섹션 추가)

감지된 기술 스택:
- TypeScript/JavaScript
- Python (FastAPI)
- Next.js 15

추가된 규칙:
- 네이밍 컨벤션
- 파일명 규칙
- Import 순서
- 에러 처리 패턴
- 보안 규칙

이제 Claude Code가 코딩 규칙을 자동으로 참조합니다.
```

## 주의사항

1. 기존 CLAUDE.md 내용 보존 (덮어쓰기 X, 추가/업데이트)
2. 프로젝트에 맞는 언어만 포함
3. 이미 있는 규칙은 중복 추가하지 않음
4. 사용자 커스텀 규칙 유지
5. **보안 라이브러리 규칙 필수 포함** - 인증/암호화 관련 코드 생성 시 반드시 검증된 라이브러리 사용
