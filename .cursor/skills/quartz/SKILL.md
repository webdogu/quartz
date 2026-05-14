---
name: quartz-blog
description: >-
  Guides Quartz 4 blog/digital garden workflows for `/home/webdogu/workspace/quartz`:
  authoring markdown under `content/`, frontmatter and drafts, local preview, GitHub
  sync, and site config. Use when the user mentions Quartz, 블로그, digital garden,
  `content/`, `npx quartz`, 배포, or publishing notes from this repo.
---

# Quartz 블로그 작성·업로드·관리 스킬

이 스킬은 **로컬 Quartz 사이트** (`/home/webdogu/workspace/quartz`) 기준이다. 작업 전 해당 디렉터리로 이동한다.

## 1. 구조 이해 (목표)

| 경로 | 역할 |
|------|------|
| `content/` | **게시할 마크다운 루트**. 하위 폴더가 URL·폴더 목록 페이지와 대응한다. |
| `quartz.config.ts` | 사이트 제목, `baseUrl`, 로케일, 플러그인(링크 해석, 초안 제외 등). |
| `quartz.layout.ts` | 헤더·사이드바(Explorer, 검색)·TOC 등 레이아웃. |
| `public/` | `npx quartz build` 결과물(정적 HTML). |

## 2. 블로그 글 작성 (방법)

1. **`content/` 아래에 `.md` 파일 추가**  
   예: `content/일기/2026-05-14-소개.md` → 사이트에서 폴더 경로가 URL에 반영된다.

2. **프론트매터(권장)**  
   파일 상단에 YAML 블록을 둔다. 예:

   ```yaml
   ---
   title: "글 제목"
   description: "목록·OG용 짧은 설명"
   tags:
     - 태그1
   draft: false
   ---
   ```

3. **초안(draft)**  
   `quartz.config.ts`에 `Plugin.RemoveDrafts()`가 있으면 **`draft: true`인 글은 빌드에서 제외**된다. 배포 전에 `draft: false`로 바꾼다.

4. **제외 폴더**  
   `configuration.ignorePatterns`에 있는 이름(예: `private`, `templates`, `.obsidian`)은 빌드 대상에서 빠진다. 비공개 노트는 `content/private/` 등으로 두는 패턴이 안전하다.

5. **내부 링크**  
   옵시디언 스타일 `[[노트이름]]`은 `Plugin.CrawlLinks`의 `markdownLinkResolution` 설정(`shortest` 등)과 맞춘다. 설정은 `quartz.config.ts`에서 변경한다.

## 3. 로컬 미리보기 (검증)

프로젝트 루트에서:

```bash
cd /home/webdogu/workspace/quartz
npx quartz build --serve
```

- 기본 포트 **8080** (필요 시 `--port`로 변경).
- **`--watch`** 를 추가하면 파일 변경 시 재빌드된다.

## 4. 업로드·배포 (GitHub 연동)

### 이 저장소의 실제 동작

- `.github/workflows/deploy.yml`은 **`v4` 브랜치에 push**될 때 `npx quartz build` 후 GitHub Pages에 올린다.
- **`git push origin sync`가 아니다.** `sync`는 브랜치 이름이 아니라 **`npx quartz sync`** 라는 **CLI 서브커맨드** 이름이다.
- 배포 트리거는 보통 아래처럼 **일반 Git 푸시**다.

```bash
cd /home/webdogu/workspace/quartz
git add -A
git status   # 확인
git commit -m "첫 배포"
git push origin v4
```

브랜치를 바꿨다면 `deploy.yml`의 `branches:` 목록과 맞출 것.

### `npx quartz sync`와의 차이

- **`npx quartz sync`**: Quartz가 안내하는 **동기화 마법사**(upstream·content 처리 등). 역시 내부에서 **Git이 원격에 접근**하므로 **인증이 필요**하다.
- **수동 `git push`**: Actions가 빌드·배포하므로 로컬에서 `public/`을 푸시할 필요는 없다. 소스만 `v4`에 올리면 된다.

### GitHub 인증 (맞는 이해)

별도 “GitHub 키” 파일이 자동으로 붙는 것은 아니고, **`git push`가 통과하려면** 아래 중 하나가 필요하다.

| 방식 | 요약 |
|------|------|
| **HTTPS** (`https://github.com/...`) | 비밀번호 대신 **Personal Access Token(PAT)** 을 쓰는 것이 일반적이다. 또는 자격 증명 도우미에 저장된 토큰. |
| **SSH** (`git@github.com:...`) | 로컬 **SSH 개인키**를 GitHub 계정에 등록한 뒤 `git remote`를 SSH URL로. |
| **GitHub CLI** | `gh auth login` 후 `gh`가 자격 증명을 관리하는 방식. |

원격이 HTTPS인지 SSH인지는 `git remote -v`로 확인한다.

### 로컬만 빌드할 때

```bash
npx quartz build
# 산출물: public/  (CI가 원격에서 다시 빌드하므로 배포 파이프라인이 Actions면 로컬 public 푸시는 보통 불필요)
```

**`quartz.config.ts`의 `baseUrl`** 은 실제 Pages URL과 맞춘다.

## 5. 일상 관리 작업

| 작업 | 방법 |
|------|------|
| 스타일·포맷 | `npm run format` |
| 타입·검사 | `npm run check` |
| Quartz 업스트림 반영 | `npx quartz update` (변경 전 커밋 권장) |
| 글 분류·URL 변경 | `content/` 안에서 폴더·파일 이동 후 링크·백링크 확인 |
| 배포 안 할 임시글 | `draft: true` 또는 `ignorePatterns`에 맞는 폴더 사용 |

## 6. 에이전트가 할 일 (체크리스트)

새 글/수정 요청 시:

- [ ] 경로가 `content/` 아래인지 확인.
- [ ] 배포 포함이면 `draft`가 `false`인지 확인.
- [ ] `title`/`description`/`tags` 등 프론트매터가 요구사항과 맞는지 확인.
- [ ] 로컬에서 `npx quartz build --serve` 또는 `npx quartz build`로 깨지지 않는지 제안.
- [ ] 배포는 사용자가 `npx quartz sync` 또는 CI 정책에 맡기고, **자격 증명이 필요한 푸시는 사용자 승인**을 받는다.

## 7. 참고

- 공식 설정: https://quartz.jzhao.xyz/configuration  
- 플러그인·폴더 페이지: `FolderPage`, `TagPage` 등은 `quartz.config.ts`의 `emitters`에 이미 포함된 경우가 많다.
