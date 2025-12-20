---
name: gitpush
user_invocable: true
description: 변경사항을 자동으로 분석하여 Conventional Commits 형식으로 커밋하고, dev 브랜치를 merge한 후 push합니다.
---

# Git Smart Push 스킬 실행 지침

사용자가 `/push` 명령어를 실행하거나 "git push해줘"라고 요청하면 다음 단계를 자동으로 수행합니다.

## 실행 단계

### 1단계: 현재 Git 상태 확인

먼저 다음 정보를 수집합니다:

```bash
# 현재 브랜치 확인
git branch --show-current

# Git 상태 확인
git status

# 커밋되지 않은 변경사항 확인
git status --porcelain

# 변경된 파일의 diff 확인
git diff

# 스테이징된 파일의 diff 확인
git diff --staged

# 모든 브랜치 확인 (dev 존재 여부)
git branch -a
```

### 2단계: 변경사항 분석 및 자동 커밋 (변경사항이 있는 경우)

**변경사항이 있는 경우** 다음을 수행합니다:

1. **변경된 파일 목록과 diff 내용 분석**
2. **Conventional Commits 형식으로 커밋 메시지 자동 생성**

**커밋 타입 결정 기준:**

- `feat:` - 새로운 기능 추가 (새 파일, 새 함수, 새 컴포넌트 등)
- `fix:` - 버그 수정 (에러 수정, 로직 수정 등)
- `docs:` - 문서 변경 (README, 주석, 문서 파일 등)
- `style:` - 코드 포맷팅, 세미콜론 누락, 공백 등
- `refactor:` - 코드 리팩토링 (기능 변경 없이 코드 개선)
- `test:` - 테스트 추가 또는 수정
- `chore:` - 빌드, 패키지 매니저, 설정 파일 변경
- `perf:` - 성능 개선

**커밋 메시지 생성 규칙:**

```
<타입>: <간결한 설명>

<상세 설명 (필요시)>

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>
```

**예시:**
```bash
# 새 기능 추가
feat: 사용자 프로필 페이지 추가

# 버그 수정
fix: 로그인 시 리다이렉트 오류 수정

# 여러 파일이 다른 타입으로 변경된 경우 주요 변경사항 기준
feat: OAuth 소셜 로그인 구현

- 카카오, 네이버, 구글 로그인 추가
- 인증 에러 처리 개선
```

3. **자동 커밋 실행:**

```bash
# 모든 변경사항 스테이징
git add .

# Conventional Commit 메시지로 커밋
git commit -m "<생성된 커밋 메시지>"
```

**변경사항이 없는 경우**: 이 단계를 건너뛰고 3단계로 진행합니다.

### 3단계: Dev 브랜치 처리 (dev 브랜치가 존재하는 경우 자동 실행)

dev 브랜치가 존재하는 경우 자동으로 다음을 수행합니다:

```bash
# 1. 현재 브랜치 저장
CURRENT_BRANCH=$(git branch --show-current)

# 2. 원격 정보 가져오기
git fetch origin

# 3. dev 브랜치가 로컬에 있는지 확인
if git show-ref --verify --quiet refs/heads/dev; then
  # 로컬 dev가 있는 경우
  git checkout dev
  git pull origin dev
else
  # 로컬 dev가 없는 경우 (원격에만 있음)
  git checkout -b dev origin/dev
  git pull origin dev
fi

# 4. 원래 브랜치로 돌아가기
git checkout $CURRENT_BRANCH

# 5. dev를 현재 브랜치에 merge
git merge dev
```

**충돌 처리**:
- Merge 중 충돌이 발생하면:
  1. 충돌 파일 목록 표시: `git diff --name-only --diff-filter=U`
  2. 사용자에게 충돌을 해결하도록 안내
  3. 스킬 실행 중단
  4. 사용자가 수동으로 충돌 해결 후 다시 `/push` 실행하도록 안내

**dev 브랜치가 없는 경우**: 이 단계를 건너뛰고 4단계로 진행합니다.

### 4단계: 현재 브랜치 Pull (자동 실행)

```bash
# 현재 브랜치 pull
git pull origin <현재-브랜치명>
```

**충돌 처리**:
- Pull 중 충돌이 발생하면 3단계와 동일하게 처리

### 5단계: Push 실행 (자동 실행)

```bash
# Push 실행
git push origin <현재-브랜치명>
```

**실패 처리**:
- Push 실패 시 에러 메시지 표시
- 일반적인 원인과 해결 방법 안내

### 6단계: 완료 메시지

성공적으로 완료되면:
```
✅ Push 완료!

작업 요약:
- 자동 커밋: {커밋 메시지} (변경사항이 있었던 경우)
- dev 브랜치 merge: {예/아니오}
- 현재 브랜치 pull: 완료
- Push 브랜치: {브랜치명}
- 최종 커밋: {커밋 해시}

원격 리포지토리가 업데이트되었습니다.
```

실패 시 명확한 에러 메시지와 다음 단계 안내를 제공합니다.

## 특수 상황 처리

### 커밋되지 않은 변경사항이 있는 경우

자동으로 다음을 수행합니다:

1. 변경된 파일 목록 출력
2. 변경사항 분석 (git diff)
3. Conventional Commits 형식으로 커밋 메시지 생성
4. 자동 커밋 (git add . && git commit)

```
📝 변경사항을 감지했습니다.

변경된 파일:
- src/app/page.tsx
- src/components/Header.tsx

변경사항을 분석하여 자동 커밋합니다...
✅ 커밋 완료: feat: 헤더 컴포넌트에 검색 기능 추가
```

### main/master 브랜치에서 실행 시

경고 메시지만 표시하고 계속 진행합니다:

```
⚠️ 경고: 현재 main 브랜치에 있습니다.
직접 push하는 것은 권장되지 않지만, 작업을 계속 진행합니다.
```

### dev 브랜치가 없는 경우

dev 브랜치 처리를 건너뛰고 바로 현재 브랜치 pull/push로 진행합니다.

## 에러 메시지 가이드

### Merge 충돌
```
❌ Merge 충돌이 발생했습니다.

충돌 파일:
{파일 목록}

해결 방법:
1. 충돌 파일을 열어서 수동으로 해결
2. git add <파일명>으로 해결된 파일 추가
3. git commit으로 merge 커밋 완료
4. 다시 /push 실행

현재 작업을 중단합니다.
```

### Push 실패
```
❌ Push가 실패했습니다.

원인: {에러 메시지}

일반적인 해결 방법:
- 원격에 새로운 커밋이 있는 경우: git pull 먼저 실행
- 권한 문제: GitHub 인증 확인
- 브랜치 보호 규칙: PR을 통해 머지 필요

다시 시도하려면 /push를 실행하세요.
```

## 사용 예시

### 예시 1: 변경사항이 있고 dev 브랜치가 있는 경우

```
User: /push

Claude:
📋 현재 Git 상태를 확인하는 중...

현재 브랜치: feature/user-profile
변경사항: 3개 파일 수정됨

📝 변경사항을 감지했습니다.

변경된 파일:
M  src/app/profile/page.tsx
M  src/components/UserCard.tsx
A  src/lib/utils/formatDate.ts

변경사항을 분석하여 자동 커밋합니다...
✅ 커밋 완료: feat: 사용자 프로필 페이지 및 날짜 포맷 유틸리티 추가

- 프로필 페이지 UI 구현
- 사용자 카드 컴포넌트 개선
- 날짜 포맷 유틸리티 함수 추가

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>

🔄 dev 브랜치를 처리하는 중...
📥 dev 브랜치 pull 완료
🔀 dev를 feature/user-profile에 merge 중...
✅ Merge 완료 (Fast-forward)

📥 현재 브랜치 pull 중...
✅ Already up to date

🚀 feature/user-profile를 push하는 중...
✅ Push 완료!

작업 요약:
- 자동 커밋: feat: 사용자 프로필 페이지 및 날짜 포맷 유틸리티 추가
- dev 브랜치 merge: 예
- 현재 브랜치 pull: 완료
- Push 브랜치: feature/user-profile
- 최종 커밋: a1b2c3d

원격 리포지토리가 업데이트되었습니다.
```

### 예시 2: 변경사항 없이 push만 하는 경우

```
User: /push

Claude:
📋 현재 Git 상태를 확인하는 중...

현재 브랜치: main
변경사항: 없음

🔄 dev 브랜치 확인 중... 없음

📥 현재 브랜치 pull 중...
✅ Already up to date

🚀 main을 push하는 중...
✅ Push 완료!

작업 요약:
- 자동 커밋: 없음 (변경사항 없음)
- dev 브랜치 merge: 아니오 (dev 없음)
- 현재 브랜치 pull: 완료
- Push 브랜치: main
- 최종 커밋: x9y8z7w

원격 리포지토리가 업데이트되었습니다.
```

### 예시 3: Merge 충돌 발생

```
User: /push

Claude:
📋 현재 Git 상태를 확인하는 중...
📝 자동 커밋 완료: fix: 로그인 버그 수정

🔄 dev 브랜치를 처리하는 중...
📥 dev 브랜치 pull 완료
🔀 dev를 feature/login에 merge 중...

❌ Merge 충돌이 발생했습니다.

충돌 파일:
- src/app/login/page.tsx
- src/lib/auth/index.ts

해결 방법:
1. 충돌 파일을 열어서 수동으로 해결
2. git add <파일명>으로 해결된 파일 추가
3. git commit으로 merge 커밋 완료
4. 다시 /push 실행

현재 작업을 중단합니다.
```

## Conventional Commits 참고

스킬은 다음 Conventional Commits 규칙을 따릅니다:

- **feat**: 새로운 기능
- **fix**: 버그 수정
- **docs**: 문서만 변경
- **style**: 코드의 의미에 영향을 주지 않는 변경사항 (공백, 포맷팅 등)
- **refactor**: 버그를 수정하거나 기능을 추가하지 않는 코드 변경
- **test**: 누락된 테스트 추가 또는 기존 테스트 수정
- **chore**: 빌드 프로세스 또는 보조 도구 및 라이브러리 변경 (문서 생성 등)
- **perf**: 성능 개선을 위한 코드 변경

변경사항이 여러 타입에 걸쳐있는 경우, 가장 중요한 변경사항의 타입을 사용합니다.
