# 개발 가이드

Diffnote 프로젝트 개발 컨벤션 및 가이드라인입니다.

## 목차

1. [프로젝트 개요](#프로젝트-개요)
2. [기술 스택](#기술-스택)
3. [프로젝트 구조](#프로젝트-구조)
4. [Git 컨벤션](#git-컨벤션)
5. [코딩 컨벤션](#코딩-컨벤션)
6. [개발 워크플로우](#개발-워크플로우)
7. [핵심 아키텍처 원칙](#핵심-아키텍처-원칙)

## 프로젝트 개요

Diffnote는 웹 기반 문서 비교 서비스입니다.

**핵심 가치:**
- 즉시 사용 가능 (회원가입/설치 불필요)
- 프라이버시 보장 (클라이언트 사이드 처리)
- 다양한 포맷 지원 (TXT, PDF, DOCX)
- 직관적인 UI (Side-by-side 비교)

**타겟 사용자:** 사무직 직장인, 법무/계약 담당자, 개발자

## 기술 스택

| 영역 | 기술 | 버전 | 선택 이유 |
|------|------|------|-----------|
| **Framework** | Next.js | 15 (App Router) | Vercel 최적화, SSR/ISR |
| **Language** | TypeScript | 최신 | 타입 안정성, 개발자 경험 |
| **Styling** | Tailwind CSS + shadcn/ui | 최신 | 빠른 개발, 일관된 디자인 시스템 |
| **PDF 파싱** | pdf.js | 최신 | 클라이언트 사이드 처리 |
| **DOCX 파싱** | mammoth.js | 최신 | 클라이언트 사이드 처리 |
| **Diff 엔진** | diff-match-patch | 최신 | Google 검증, 성능 우수 |
| **상태관리** | Zustand | 최신 | 경량, 간단한 API |
| **배포** | Vercel | - | Git 연동 자동 배포 |
| **DB** | Vercel KV / Postgres | - | 공유 링크 저장 (Phase 2) |

## 프로젝트 구조

```
diffnote/
├── app/                      # Next.js App Router
│   ├── page.tsx              # 메인 비교 페이지
│   ├── layout.tsx            # 루트 레이아웃
│   ├── shared/[id]/page.tsx  # 공유 페이지 (Phase 2)
│   └── api/share/route.ts    # 공유 API (Phase 2)
├── components/               # React 컴포넌트
│   ├── DiffViewer.tsx        # 비교 결과 UI
│   ├── FileUploader.tsx      # 파일 업로드
│   └── TextInput.tsx         # 텍스트 입력
├── lib/                      # 유틸리티 및 핵심 로직
│   ├── parsers/              # PDF, DOCX 파서
│   ├── diff-engine.ts        # 비교 로직
│   └── utils.ts              # 공통 유틸리티
├── public/                   # 정적 파일
└── docs/                     # 프로젝트 문서
    ├── PRD.md                # 제품 요구사항 문서
    └── CONTRIBUTING.md       # 개발 가이드 (본 문서)
```

## Git 컨벤션

### 커밋 메시지 규칙

**형식:**
```
<type>: <subject>

<body (선택)>
```

**Type 분류:**

| Type | 설명 | 예시 |
|------|------|------|
| `feat` | 새로운 기능 추가 | `feat: 텍스트 비교 기능 구현` |
| `fix` | 버그 수정 | `fix: PDF 파싱 오류 수정` |
| `docs` | 문서 수정 | `docs: README 업데이트` |
| `style` | 코드 포맷팅, 세미콜론 누락 등 | `style: Prettier 적용` |
| `refactor` | 코드 리팩토링 (기능 변경 없음) | `refactor: DiffViewer 컴포넌트 분리` |
| `perf` | 성능 개선 | `perf: diff 알고리즘 최적화` |
| `test` | 테스트 코드 추가/수정 | `test: FileUploader 단위 테스트 추가` |
| `chore` | 빌드, 패키지 매니저 설정 등 | `chore: dependencies 업데이트` |

**작성 규칙:**

✅ **DO**
- 제목은 한글로 작성
- 제목은 50자 이내로 간결하게
- 제목 끝에 마침표(`.`) 금지
- 본문이 필요한 경우 제목과 본문 사이 빈 줄 삽입
- 본문은 "무엇을", "왜" 변경했는지 설명

❌ **DON'T**
- 영어와 한글 혼용 지양
- 모호한 표현 사용 금지 (예: "수정", "변경")

**예시:**

```bash
# 좋은 예
feat: Side-by-Side 비교 뷰 구현

- 좌우 분할 레이아웃 추가
- 줄 단위 비교 결과 시각화
- 변경사항 하이라이트 기능 (추가/삭제/수정)

# 나쁜 예
feat: 기능 추가
```

### 브랜치 전략

**1인 개발이므로 PR 없이 바로 메인 브랜치에 커밋**

- `main`: 프로덕션 배포 브랜치
- 필요시 기능 브랜치 생성 후 작업 완료 시 즉시 머지

### 커밋 후 즉시 Push

- 커밋 완료 후 바로 `git push` 실행
- 로컬에 커밋을 쌓아두지 않음

## 코딩 컨벤션

### TypeScript

- **타입 명시**: 모든 함수 파라미터와 반환값에 타입 명시
- **any 금지**: `any` 타입 사용 지양, `unknown` 또는 명시적 타입 사용
- **타입 별칭**: 재사용되는 타입은 별도 타입으로 정의

```typescript
// ✅ Good
type DiffResult = {
  added: string[];
  removed: string[];
  modified: string[];
};

function compareDocs(doc1: string, doc2: string): DiffResult {
  // ...
}

// ❌ Bad
function compareDocs(doc1, doc2) {
  // ...
}
```

### React 컴포넌트

- **함수형 컴포넌트**: 클래스형 컴포넌트 사용 금지
- **네이밍**: PascalCase 사용 (예: `DiffViewer.tsx`)
- **Props 타입**: 모든 컴포넌트는 Props 인터페이스 정의

```typescript
// ✅ Good
interface DiffViewerProps {
  leftText: string;
  rightText: string;
  mode: 'side-by-side' | 'unified';
}

export default function DiffViewer({ leftText, rightText, mode }: DiffViewerProps) {
  // ...
}
```

### 파일 네이밍

- **컴포넌트**: PascalCase (예: `FileUploader.tsx`)
- **유틸리티**: kebab-case (예: `diff-engine.ts`)
- **상수**: UPPER_SNAKE_CASE (예: `const MAX_FILE_SIZE = 10 * 1024 * 1024;`)

### Tailwind CSS

- **클래스 순서**: Prettier Plugin for Tailwind CSS 사용
- **커스텀 클래스**: 최소화, Tailwind 유틸리티 우선 사용
- **반응형**: 모바일 퍼스트 (기본 → `sm:` → `md:` → `lg:`)

## 개발 워크플로우

### 1. 로컬 개발 환경 설정

```bash
# 저장소 클론
git clone https://github.com/vivajayyy/diffnote.git
cd diffnote

# 의존성 설치
npm install

# 개발 서버 실행
npm run dev
```

### 2. 기능 개발 프로세스

1. **이슈 확인**: PRD 또는 로드맵 확인
2. **개발**: 로컬에서 기능 구현
3. **테스트**: 브라우저에서 수동 테스트
4. **커밋**: Git 컨벤션에 맞춰 커밋
5. **푸시**: 즉시 원격 저장소에 푸시
6. **배포**: Vercel 자동 배포 확인

### 3. 빌드 및 배포

```bash
# 프로덕션 빌드
npm run build

# 로컬에서 프로덕션 빌드 실행
npm run start

# Vercel 배포 (자동)
git push origin main
```

## 핵심 아키텍처 원칙

### 1. 클라이언트 중심 처리

> **모든 파일 처리는 브라우저에서 수행합니다.**

- 서버에 파일을 업로드하지 않음
- PDF/DOCX 파싱은 `pdf.js`, `mammoth.js` 사용
- Diff 비교는 `diff-match-patch` 사용

**이유:**
- 프라이버시 보장
- Vercel Serverless 제약 회피
- 비용 절감

### 2. 프라이버시 우선

- 사용자 파일은 서버에 저장하지 않음
- 공유 기능 구현 시에도 결과만 저장 (원본 파일 미저장)
- 로컬 스토리지 활용 (비교 이력)

### 3. 성능 최적화

- **Web Worker**: 대용량 파일 처리 시 메인 스레드 블로킹 방지
- **Lazy Loading**: 컴포넌트 코드 스플리팅
- **Memoization**: React.memo, useMemo 적극 활용

### 4. 접근성 (a11y)

- 키보드 내비게이션 지원
- ARIA 속성 적절히 사용
- 색상 외 추가 시각적 표시 (아이콘, 패턴)

## Phase별 개발 우선순위

### Phase 1: MVP (현재)

**목표:** 기본 텍스트/파일 비교 기능

- [ ] 텍스트 입력 UI
- [ ] 파일 업로드 (TXT, PDF, DOCX)
- [ ] Diff 엔진 연동
- [ ] Side-by-Side 결과 뷰
- [ ] Vercel 배포

### Phase 2: 확장 기능

- [ ] 공유 링크 생성
- [ ] 다크모드
- [ ] 결과 내보내기 (HTML/PDF)
- [ ] 비교 이력 관리

### Phase 3: 프리미엄

- [ ] 3-Way Merge
- [ ] API 제공
- [ ] 팀 워크스페이스
- [ ] 결제 시스템

## 참고 자료

- [PRD 문서](./PRD.md)
- [Next.js 공식 문서](https://nextjs.org/docs)
- [Tailwind CSS 문서](https://tailwindcss.com/docs)
- [shadcn/ui 문서](https://ui.shadcn.com/)
- [diff-match-patch GitHub](https://github.com/google/diff-match-patch)

---

**마지막 업데이트:** 2026년 1월 7일
