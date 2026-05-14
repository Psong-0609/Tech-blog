# Tech 블로그 MVP 로드맵

목표: 정적·파일 기반 글로 블로그를 빠르게 출시하고, **글 작성은 저장소의 Markdown 파일 추가·커밋으로만** 진행한다. 방문자용 UI에는 로그인·에디터가 없다.

현재 스택: Next.js 16(App Router), React 19, Emotion, TypeScript.

---

## 원칙

| 항목       | 결정                                                                     |
| ---------- | ------------------------------------------------------------------------ |
| 글 소스    | `content/posts/` 등 디렉터리의 `.md` 파일                                |
| 메타데이터 | 각 글 상단 YAML front matter (`title`, `date`, `description`, `slug` 등) |
| 작성 권한  | Git 저장소에 푸시할 수 있는 사람만 → MVP에서는 본인만 초대하면 충분      |
| 빌드 시점  | `next build` 때 MD를 읽어 목록·본문 생성 (별도 DB 없음)                  |

---

## Phase 1 — 컴포넌트 & 디자인 기반

**목표:** 페이지에서 재사용할 레이아웃·타이포·카드 UI를 고정한다.

**디자인 상세(색·타이포·간격·컴포넌트 외형)**는 [`design.md`](./design.md)에서 관리한다.

1. **토큰 반영**
   - `design.md`의 토큰을 CSS 변수 또는 Emotion theme 한곳에 매핑한다.
   - `layout.tsx`에서 기본 `lang`을 `ko`로 둔다.

2. **공통 레이아웃 컴포넌트**
   - `Header`: 블로그 타이틀 + 네비(선택: About 등 단일 링크만 MVP면 생략 가능).
   - `Footer`: 저작권·간단 링크.
   - `Container`: 좌우 패딩·최대 너비 제한.

3. **콘텐츠용 컴포넌트**
   - `PostCard`: 목록용 — 제목, 요약, 날짜.
   - `Prose`(또는 `MarkdownBody`): 상세 본문용 — 제목·단락·코드·인용 등 마크다운 렌더 결과 스타일링.

4. **완료 기준**
   - `design.md`에 확정한 테마·토큰만 참조하고, 메인·상세에서 동일 규칙을 쓴다.

---

## Phase 2 — 메인 페이지

**목표:** 글 목록을 시간 역순으로 보여준다.

1. **데이터 로딩**
   - `src/util/posts.ts`(명칭 자유): `content/posts/*.md`를 읽고 front matter를 파싱해 `{ slug, title, date, description }[]` 반환.
   - 빌드 타임 전용(`fs` 사용) → Server Component에서 호출.

2. **UI**
   - Phase 1의 `Container` + `PostCard`로 리스트 렌더.
   - 글이 없을 때 빈 상태 메시지.

3. **완료 기준**
   - 로컬에 샘플 MD 2~3개 두고 목록이 정상 표시된다.

---

## Phase 3 — 상세 페이지

**목표:** `/posts/[slug]`에서 해당 글 전체를 읽는다.

1. **라우팅**
   - `src/app/posts/[slug]/page.tsx`
   - `generateStaticParams`: 존재하는 모든 slug로 정적 경로 생성.

2. **Markdown → HTML (또는 React)**
   - 의존성 예: `gray-matter`(front matter), `remark` / `rehype` 계열 또는 `next-mdx-remote` 등 팀이 선호하는 한 줄기로 통일.
   - 코드 하이라이트가 필요하면 이 단계 또는 직후에만 추가(범위 확대 주의).

3. **SEO·메타**
   - `generateMetadata`로 `title`, `description`, canonical(선택) 설정.

4. **완료 기준**
   - 목록에서 카드 클릭 → 상세 이동, 본문·제목·날짜 일치.

---

## Phase 4 — 오로지 나만 Markdown으로 글 작성

**목표:** 프로덕션에서 **방문자는 글을 쓸 수 없고**, 배포 파이프라인과 저장소 접근으로만 글이 바뀐다.

1. **워크플로우**
   - 새 글: `content/posts/yyyy-mm-dd-slug.md` 생성 → front matter 작성 → 커밋 → 푸시 → CI/CD가 빌드.

2. **저장소·호스팅**
   - GitHub 등 비공개 저장소 또는 협력자 목록에서 본인만 유지.
   - Vercel 등 배포 시 Production 브랜치 보호.

3. **앱 레벨에서 하지 않을 것 (MVP)**
   - 관리자 로그인, 브라우저 에디터, API로 글 저장 — 필요 없음.

4. **완료 기준**
   - 문서화된 “새 글 추가 방법” 한 페이지(이 파일의 요약 또는 `README` 한 단락)가 있고, 그 절차만으로 글이 반영된다.

---

## 권장 진행 순서 (체크리스트)

- [ ] Phase 1: 토큰· globals· Header / Footer / Container· PostCard· Prose
- [ ] Phase 2: `src/util/posts` + 메인 페이지 목록
- [ ] Phase 3: `[slug]` 상세 + 정적 생성 + 메타데이터
- [ ] Phase 4: 샘플 글·폴더 규칙 확정 + 배포·권한 점검

---

## 다음 단계 (MVP 이후)

검색, 태그, RSS, OG 이미지, 다크 모드 토글, 읽는 시간 표시 등은 사용자 반응을 본 뒤 순차 추가한다.

---

## 참고: 예상 의존성 추가 시점

| 단계                        | 패키지 예시                              |
| --------------------------- | ---------------------------------------- |
| Phase 2~3                   | `gray-matter`, 마크다운 파이프라인 1세트 |
| Phase 3 (코드 블록 강조 시) | `shiki` 또는 `rehype-pretty-code` 등     |

구체 패키지 버전은 구현 시점의 Next·React와 호환 표를 보고 `package.json`에 고정한다.
