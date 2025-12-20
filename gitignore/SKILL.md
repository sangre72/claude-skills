# Gitignore 생성 스킬

사용자가 `/gitignore` 명령어를 실행하면 프로젝트에 맞는 포괄적인 `.gitignore` 파일을 생성합니다.

## 실행 조건

- 사용자가 `/gitignore` 실행
- "gitignore 만들어줘" 또는 "gitignore 생성" 요청

## 기본 템플릿

```gitignore
# See https://help.github.com/articles/ignoring-files/ for more about ignoring files.

# dependencies
node_modules/
/.pnp
.pnp.js
.yarn/install-state.gz

# testing
/coverage
__tests__/coverage/

# next.js
.next/
/.next/
/out/
apps/*/.next/
packages/*/.next/

# turbo
.turbo/
apps/*/.turbo/
packages/*/.turbo/

# build & production
build/
.build/
dist/
out/
*.build/

# misc
.DS_Store
*.pem
Thumbs.db

# debug & logs
npm-debug.log*
yarn-debug.log*
yarn-error.log*
pnpm-debug.log*
*.log

# local env files
.env
.env.*
.env*.local
!.env.example

# vercel
.vercel

# typescript
*.tsbuildinfo
next-env.d.ts

# python
__pycache__/
*.py[cod]
*$py.class
*.so
.Python
.venv/
venv/
env/
ENV/
.env/
backend/.venv/
**/site-packages/
*.egg
*.egg-info/
dist/
eggs/
.eggs/
*.whl
pip-wheel-metadata/
.pytest_cache/
.mypy_cache/
.ruff_cache/

# IDE
.vscode/
.idea/
*.swp
*.swo
*.sublime-*
*.code-workspace

# Claude Code
.claude/
settings.json

# cache
.cache/
*.cache
.parcel-cache/
.eslintcache
.stylelintcache

# temp files
tmp/
temp/
*.tmp
*.temp
```

## 실행 단계

### 1단계: 프로젝트 분석

프로젝트 루트에서 다음을 확인합니다:

```bash
# 기존 .gitignore 확인
cat .gitignore 2>/dev/null || echo "없음"

# 프로젝트 타입 확인
ls package.json pyproject.toml requirements.txt Cargo.toml go.mod 2>/dev/null
```

### 2단계: 프로젝트별 추가 패턴

#### Node.js/JavaScript 프로젝트
- `node_modules/`, `.next/`, `.turbo/`, `dist/`, `build/`

#### Python 프로젝트
- `__pycache__/`, `.venv/`, `venv/`, `*.pyc`, `.pytest_cache/`, `.mypy_cache/`

#### Monorepo (Turborepo/pnpm)
- `apps/*/.next/`, `apps/*/.turbo/`, `packages/*/.next/`

#### Go 프로젝트
- `vendor/`, `*.exe`, `*.test`

#### Rust 프로젝트
- `target/`, `Cargo.lock` (라이브러리인 경우)

### 3단계: 파일 생성 또는 업데이트

1. 기존 `.gitignore`가 있으면 누락된 패턴 추가
2. 없으면 새로 생성
3. 중복 패턴 제거

### 4단계: 이미 추적 중인 파일 처리

gitignore에 추가된 파일이 이미 git에 추적 중인 경우:

```bash
# 추적 중인 파일 확인
git ls-files | grep -E "(node_modules|\.next|\.turbo|__pycache__|\.venv)"

# 추적 해제 (필요시)
git rm -r --cached <경로>
```

## 완료 메시지

```
✅ .gitignore 생성/업데이트 완료!

추가된 패턴:
- Node.js: node_modules/, .next/, dist/
- Python: __pycache__/, .venv/, *.pyc
- Build: build/, .build/, out/
- IDE: .vscode/, .idea/
- Claude: .claude/, settings.json

⚠️ 이미 추적 중인 파일이 있다면:
git rm -r --cached <경로> 로 추적 해제하세요.
```

## 주의사항

1. `.env.example`은 gitignore에서 제외 (`!.env.example`)
2. `settings.json`은 IDE/에디터 설정이므로 제외
3. `.claude/`는 개인 설정이므로 제외
4. `node_modules/`, `.venv/`는 의존성이므로 반드시 제외
