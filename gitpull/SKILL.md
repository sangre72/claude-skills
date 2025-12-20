---
name: gitpull
user_invocable: true
description: dev 브랜치가 있으면 pull 후 현재 브랜치에 merge하고, 현재 브랜치도 pull합니다.
---

# Git Smart Pull 스킬 실행 지침

사용자가 `/gitpull` 명령어를 실행하거나 "git pull해줘", "최신 코드 받아줘"라고 요청하면 다음 단계를 자동으로 수행합니다.

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

# 모든 브랜치 확인 (dev 존재 여부)
git branch -a

# 원격 정보 가져오기
git fetch origin
```

### 2단계: 커밋되지 않은 변경사항 처리

**변경사항이 있는 경우**:

```
⚠️ 커밋되지 않은 변경사항이 있습니다.

변경된 파일:
{파일 목록}

다음 중 하나를 선택하세요:
1. 변경사항을 stash하고 계속 진행 (권장)
2. 먼저 /gitpush로 커밋 후 다시 시도
3. 작업 중단

선택: [1]
```

**Stash 선택 시**:
```bash
git stash push -m "auto-stash before gitpull"
```

**변경사항이 없는 경우**: 3단계로 진행합니다.

### 3단계: Dev 브랜치 처리 (dev 브랜치가 존재하는 경우)

dev 브랜치가 존재하는 경우 자동으로 다음을 수행합니다:

```bash
# 1. 현재 브랜치 저장
CURRENT_BRANCH=$(git branch --show-current)

# 2. dev 브랜치가 로컬에 있는지 확인
if git show-ref --verify --quiet refs/heads/dev; then
  # 로컬 dev가 있는 경우
  git checkout dev
  git pull origin dev
else
  # 로컬 dev가 없는 경우 (원격에만 있음)
  git checkout -b dev origin/dev
fi

# 3. 원래 브랜치로 돌아가기
git checkout $CURRENT_BRANCH

# 4. dev를 현재 브랜치에 merge
git merge dev --no-edit
```

**충돌 처리**:
- Merge 중 충돌이 발생하면:
  1. 충돌 파일 목록 표시: `git diff --name-only --diff-filter=U`
  2. 사용자에게 충돌을 해결하도록 안내
  3. 스킬 실행 중단
  4. 충돌 해결 후 다시 `/gitpull` 실행하도록 안내

**dev 브랜치가 없는 경우**: 이 단계를 건너뛰고 4단계로 진행합니다.

### 4단계: 현재 브랜치 Pull

```bash
# 현재 브랜치 pull
git pull origin <현재-브랜치명>
```

**충돌 처리**:
- Pull 중 충돌이 발생하면 3단계와 동일하게 처리

### 5단계: Stash 복원 (stash했던 경우)

```bash
# Stash 복원
git stash pop
```

**충돌 발생 시**:
```
⚠️ Stash 적용 중 충돌이 발생했습니다.

충돌 파일:
{파일 목록}

수동으로 충돌을 해결한 후:
1. git add <파일명>
2. git stash drop  # stash 제거
```

### 6단계: 완료 메시지

성공적으로 완료되면:
```
✅ Pull 완료!

작업 요약:
- 현재 브랜치: {브랜치명}
- dev 브랜치 merge: {예/아니오}
- 현재 브랜치 pull: 완료
- Stash 복원: {예/아니오/해당없음}
- 최신 커밋: {커밋 해시} - {커밋 메시지}

로컬 브랜치가 최신 상태입니다.
```

## 특수 상황 처리

### 현재 브랜치가 dev인 경우

dev 브랜치에서 실행하면 merge 단계를 건너뜁니다:

```
📋 현재 dev 브랜치입니다.
dev 자체를 pull합니다...

✅ Pull 완료!
- dev 브랜치 pull: 완료
- 최신 커밋: {커밋 해시}
```

### 현재 브랜치가 main/master인 경우

경고 메시지만 표시하고 계속 진행합니다:

```
⚠️ 경고: 현재 main 브랜치에 있습니다.
```

### 원격 브랜치가 없는 경우

```
ℹ️ 원격에 현재 브랜치({브랜치명})가 없습니다.
dev 브랜치 merge만 수행합니다.
```

### dev 브랜치도 없고 원격 브랜치도 없는 경우

```
ℹ️ Pull할 원격 브랜치가 없습니다.
현재 로컬 상태를 유지합니다.
```

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
4. 다시 /gitpull 실행 (또는 작업 계속)

현재 작업을 중단합니다.
```

### Pull 실패
```
❌ Pull이 실패했습니다.

원인: {에러 메시지}

일반적인 해결 방법:
- 네트워크 연결 확인
- GitHub 인증 확인
- 원격 리포지토리 접근 권한 확인

다시 시도하려면 /gitpull을 실행하세요.
```

### Stash 충돌
```
⚠️ Stash 적용 중 충돌이 발생했습니다.

pull은 성공적으로 완료되었지만, 이전 변경사항을 복원하는 중 충돌이 발생했습니다.

충돌 파일:
{파일 목록}

해결 방법:
1. 충돌 파일을 열어서 수동으로 해결
2. git add <파일명>
3. git stash drop  # stash 제거

주의: 로컬 브랜치는 이미 최신 상태입니다.
```

## 사용 예시

### 예시 1: 정상적인 Pull (dev 있음)

```
User: /gitpull

Claude:
📋 현재 Git 상태를 확인하는 중...

현재 브랜치: feature/user-profile
변경사항: 없음

🔄 원격 정보를 가져오는 중...
✅ Fetch 완료

🔄 dev 브랜치를 처리하는 중...
📥 dev 브랜치로 전환...
📥 dev 브랜치 pull 완료 (3 commits)
🔙 feature/user-profile로 복귀...
🔀 dev를 feature/user-profile에 merge 중...
✅ Merge 완료 (Fast-forward)

📥 현재 브랜치 pull 중...
✅ feature/user-profile pull 완료 (Already up to date)

✅ Pull 완료!

작업 요약:
- 현재 브랜치: feature/user-profile
- dev 브랜치 merge: 예 (3 commits)
- 현재 브랜치 pull: 완료
- Stash 복원: 해당없음
- 최신 커밋: a1b2c3d - feat: 사용자 프로필 기능 추가

로컬 브랜치가 최신 상태입니다.
```

### 예시 2: 변경사항이 있는 경우

```
User: /gitpull

Claude:
📋 현재 Git 상태를 확인하는 중...

현재 브랜치: feature/login
변경사항: 2개 파일 수정됨

⚠️ 커밋되지 않은 변경사항이 있습니다.

변경된 파일:
M  src/app/login/page.tsx
M  src/lib/auth/index.ts

변경사항을 stash하고 계속 진행합니다...
📦 Stash 완료: auto-stash before gitpull

🔄 dev 브랜치를 처리하는 중...
... (중간 과정 생략)

📦 Stash를 복원하는 중...
✅ Stash 복원 완료

✅ Pull 완료!

작업 요약:
- 현재 브랜치: feature/login
- dev 브랜치 merge: 예
- 현재 브랜치 pull: 완료
- Stash 복원: 예 (2개 파일)
- 최신 커밋: x9y8z7w - fix: 로그인 오류 수정

로컬 브랜치가 최신 상태입니다.
```

### 예시 3: dev 브랜치가 없는 경우

```
User: /gitpull

Claude:
📋 현재 Git 상태를 확인하는 중...

현재 브랜치: main
변경사항: 없음

🔄 원격 정보를 가져오는 중...
✅ Fetch 완료

ℹ️ dev 브랜치가 없습니다. 건너뜁니다.

📥 현재 브랜치 pull 중...
✅ main pull 완료 (2 commits behind -> up to date)

✅ Pull 완료!

작업 요약:
- 현재 브랜치: main
- dev 브랜치 merge: 아니오 (dev 없음)
- 현재 브랜치 pull: 완료 (2 commits)
- Stash 복원: 해당없음
- 최신 커밋: abc1234 - chore: 의존성 업데이트

로컬 브랜치가 최신 상태입니다.
```

### 예시 4: Merge 충돌 발생

```
User: /gitpull

Claude:
📋 현재 Git 상태를 확인하는 중...

현재 브랜치: feature/auth
변경사항: 없음

🔄 dev 브랜치를 처리하는 중...
📥 dev 브랜치 pull 완료
🔀 dev를 feature/auth에 merge 중...

❌ Merge 충돌이 발생했습니다.

충돌 파일:
- src/lib/auth/errors.ts
- src/app/api/auth/callback/route.ts

해결 방법:
1. 충돌 파일을 열어서 수동으로 해결
   - <<<<<<< 와 >>>>>>> 사이의 코드 정리
2. git add src/lib/auth/errors.ts src/app/api/auth/callback/route.ts
3. git commit -m "merge: resolve conflicts with dev"
4. 필요시 다시 /gitpull 실행

현재 작업을 중단합니다.
```

## 요약 플로우차트

```
/gitpull 실행
    │
    ▼
┌─────────────────┐
│ 1. 상태 확인    │
└────────┬────────┘
         │
         ▼
    변경사항 있음? ──예──▶ Stash 저장
         │                    │
        아니오                │
         │◀───────────────────┘
         ▼
┌─────────────────┐
│ 2. git fetch    │
└────────┬────────┘
         │
         ▼
    dev 있음? ──예──▶ checkout dev → pull → checkout back → merge
         │                                                    │
        아니오                                                │
         │◀───────────────────────────────────────────────────┘
         │                              충돌? ──예──▶ 중단 & 안내
         ▼
┌─────────────────┐
│ 3. 현재 브랜치  │
│    pull         │
└────────┬────────┘
         │                              충돌? ──예──▶ 중단 & 안내
         ▼
    Stash 있음? ──예──▶ git stash pop
         │                    │
        아니오                │
         │◀───────────────────┘
         │                              충돌? ──예──▶ 경고 & 수동 해결 안내
         ▼
┌─────────────────┐
│ 4. 완료 메시지  │
└─────────────────┘
```
