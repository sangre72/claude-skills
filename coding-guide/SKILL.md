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
