# 모듈화 검사 스킬

사용자가 `/modular-check` 명령어를 실행하면 프로젝트의 모듈화 상태를 분석하고 CLAUDE.md에 아키텍처 가이드를 추가합니다.

## 증분 분석 원칙

**IMPORTANT**: CLAUDE.md에 소스가 추가되거나 수정된 경우, 전체 프로젝트가 아닌 **추가/변경된 부분만** 분석합니다.

- 새 모듈 추가 시 → 해당 모듈의 구조만 검사
- 타입 파일 수정 시 → 해당 타입의 위치/중복만 검사
- 의존성 변경 시 → 변경된 import 관계만 검사

```bash
# 최근 변경 파일 확인
git diff --name-only HEAD~1

# 변경된 파일만 대상으로 분석
```

## 실행 조건

- 사용자가 `/modular-check` 실행
- "모듈화 검사해줘", "구조 분석해줘" 요청

## 실행 단계

### 1단계: 프로젝트 구조 분석

```bash
# 디렉토리 구조 확인
tree -L 3 -d 2>/dev/null || find . -type d -maxdepth 3

# 패키지 구조 확인 (monorepo)
ls apps/ packages/ 2>/dev/null

# 타입 파일 위치 확인
find . -name "*.ts" -path "*/types/*" 2>/dev/null
find . -name "types.ts" -o -name "*.types.ts" 2>/dev/null
```

### 2단계: 모듈화 검사 항목

#### 1. 타입 중복 검사 (30%)
```bash
# 동일한 인터페이스가 여러 파일에 정의되어 있는지
grep -r "interface PaginatedResponse" --include="*.ts" 2>/dev/null
grep -r "interface SuccessResponse" --include="*.ts" 2>/dev/null
grep -r "interface User" --include="*.ts" 2>/dev/null
```

- ✅ 공유 타입은 `packages/shared/src/types/`에만 정의
- ❌ 앱별로 중복 정의된 타입 발견

#### 2. 순환 의존성 검사 (20%)
```bash
# A → B → A 형태의 import 순환 확인
# 패키지 간 상호 참조 확인
```

- ✅ 단방향 의존성 유지
- ❌ 순환 참조 발견

#### 3. 레이어 분리 검사 (20%)
```
lib/       → 비즈니스 로직 (순수 함수)
components/ → UI 컴포넌트
hooks/     → React 훅
types/     → 타입 정의
utils/     → 유틸리티 함수
```

- ✅ 레이어별 책임 분리
- ❌ 컴포넌트에 비즈니스 로직 혼재

#### 4. 앱별 타입 분리 (15%)
```
apps/admin/src/types/  → 관리자 전용 타입
apps/user/src/types/   → 사용자 전용 타입
packages/shared/       → 공유 타입
```

- ✅ 앱 전용 타입이 types/ 디렉토리에 분리
- ❌ hooks 파일 내에 타입 정의

#### 5. Public API 정의 (15%)
```
packages/shared/src/index.ts  → 명시적 export
packages/ui/src/index.ts      → 명시적 export
```

- ✅ index.ts에서 public API만 export
- ❌ 내부 모듈 직접 import

### 3단계: 준수율 계산

```
모듈화 준수율 = (통과 항목 / 전체 항목) × 100

예시:
- 타입 중복 없음: 30%
- 순환 의존성 없음: 20%
- 레이어 분리 완료: 20%
- 앱별 타입 분리: 15%
- Public API 정의: 15%
총 준수율: 100%
```

### 4단계: CLAUDE.md에 아키텍처 가이드 추가

```markdown
## 아키텍처 가이드

### 모듈화 원칙
1. **공유 타입**: `packages/shared/src/types/`에만 정의
2. **앱 전용 타입**: `apps/*/src/types/`에 분리
3. **레이어 분리**: lib(로직), components(UI), hooks(상태)
4. **Public API**: `index.ts`에서 명시적 export

### 디렉토리 구조
```
packages/
├── shared/          # 공유 타입, 유틸리티
│   └── src/types/   # 공통 타입 정의
└── ui/              # 공유 UI 컴포넌트

apps/
├── admin/
│   └── src/
│       ├── types/   # 관리자 전용 타입
│       ├── lib/     # 비즈니스 로직
│       └── components/
└── user/
    └── src/
        ├── types/   # 사용자 전용 타입
        ├── lib/
        └── components/
```

### 의존성 규칙
- apps → packages (O)
- packages → apps (X)
- shared → ui (X)
- ui → shared (O)
```

### 5단계: 개선 필요 사항 리포트

발견된 문제점을 CLAUDE.md에 TODO로 추가:

```markdown
## 리팩터링 TODO

### 모듈화 개선 필요 (현재 준수율: XX%)
- [ ] `apps/user/src/hooks/useQueue.ts` 내 타입을 `types/queue.ts`로 분리
- [ ] `apps/admin/src/lib/api.ts`의 중복 타입 제거 → `@auction/shared` 사용
- [ ] 순환 참조 해결: `components/A.tsx` ↔ `components/B.tsx`
```

## 완료 메시지

```
✅ 모듈화 검사 완료!

📊 모듈화 준수율: 85%

검사 결과:
✅ 타입 중복 검사: 통과 (30%)
✅ 순환 의존성: 통과 (20%)
✅ 레이어 분리: 통과 (20%)
⚠️ 앱별 타입 분리: 일부 미흡 (10/15%)
✅ Public API: 통과 (15%)

발견된 이슈:
1. apps/user/src/hooks/useQueue.ts
   - QueueViewer, QueueListData 타입이 파일 내 정의됨
   - 권장: apps/user/src/types/queue.ts로 분리

업데이트된 파일:
- CLAUDE.md (아키텍처 가이드 추가)

/refactor 스킬로 자동 수정할 수 있습니다.
```

## 주의사항

1. 코드 수정 없이 분석만 수행
2. 기존 CLAUDE.md 내용 보존
3. 구체적인 파일 경로와 라인 번호 제공
4. /refactor 스킬과 연계하여 수정 안내
