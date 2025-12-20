# Claude Code Skills Collection

[English](./README.md)

Claude Code에서 사용할 수 있는 커스텀 스킬 모음입니다.

## 설치

```bash
git clone https://github.com/sangre72/claude-skills.git ~/.claude/skills
```

## 스킬 목록

| 스킬 | 명령어 | 설명 |
|------|--------|------|
| [gitpush](./gitpush) | `/gitpush` | 변경사항을 자동 분석하여 Conventional Commits 형식으로 커밋하고, dev 브랜치 merge 후 push |
| [gitpull](./gitpull) | `/gitpull` | dev 브랜치가 있으면 pull 후 현재 브랜치에 merge하고, 현재 브랜치도 pull |
| [coding-guide](./coding-guide) | `/coding-guide` | 프로젝트에 맞는 코딩 가이드라인을 생성하여 CLAUDE.md에 추가 |
| [gitignore](./gitignore) | `/gitignore` | 프로젝트에 맞는 포괄적인 .gitignore 파일 생성 |
| [modular-check](./modular-check) | `/modular-check` | 프로젝트의 모듈화 상태를 분석하고 아키텍처 가이드 제공 |
| [refactor](./refactor) | `/refactor` | 모듈화 및 타입 가이드라인 준수 여부 검사 및 리팩토링 |

## 스킬 상세

### /gitpush

변경사항을 자동으로 분석하여 Conventional Commits 형식으로 커밋합니다.

- 변경된 파일 분석 후 적절한 커밋 타입 결정 (feat, fix, docs, refactor 등)
- dev 브랜치가 있으면 자동으로 merge
- 현재 브랜치 pull 후 push

### /gitpull

dev 브랜치와 현재 브랜치를 동기화합니다.

- 커밋되지 않은 변경사항이 있으면 자동 stash
- dev 브랜치가 있으면 pull 후 현재 브랜치에 merge
- 현재 브랜치 pull
- stash 복원

### /coding-guide

프로젝트에 맞는 코딩 가이드라인을 생성합니다.

- TypeScript/JavaScript, Python 등 언어별 규칙
- 네이밍 컨벤션, 파일명 규칙
- Import 순서, 에러 처리 패턴
- 보안 라이브러리 규칙

### /gitignore

프로젝트에 맞는 .gitignore 파일을 생성합니다.

- Node.js, Python, Go, Rust 등 다양한 언어 지원
- Monorepo (Turborepo/pnpm) 지원
- IDE, 환경 변수, 캐시 파일 등 포함

### /modular-check

프로젝트의 모듈화 상태를 분석합니다.

- 타입 중복 검사
- 순환 의존성 검사
- 레이어 분리 검사
- 모듈화 준수율 계산

### /refactor

모듈화 및 타입 가이드라인 준수 여부를 검사합니다.

- 타입 통합 (중복 타입 제거)
- 유틸리티 통합
- 의존성 방향 검사
- 자동 수정 지원 (`--fix`)

## 라이선스

MIT
