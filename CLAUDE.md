# 아띠 (Atti) — CLAUDE.md

## 1. 프로젝트 소개

아띠는 **감정 일기 앱**이다. 사용자가 일기를 쓰면 AI가 감정을 분석하고, 공감 문장과 작은 행동(음악, 호흡, 글쓰기 등)을 추천한다. 위기 신호가 감지되면 안전 안내 화면으로 연결한다.

### 사용 기술 (코드 기준)

- **프레임워크**: Next.js 16 (App Router) + React 19 + TypeScript
- **스타일**: Tailwind CSS 4 (`@tailwindcss/postcss`), shadcn/ui 방식 설정(`components.json`), `clsx`/`tailwind-merge`/`class-variance-authority`
- **AI**: Google Gemini (`@google/generative-ai`), 환경 변수 `GOOGLE_API_KEY`
- **검증**: `zod` (AI 응답 형식 검증)
- **차트/날짜/아이콘**: `recharts`, `date-fns`, `lucide-react`
- **데이터 저장**: 서버 DB 없음. 브라우저 `localStorage`에 저장 (`lib/storage.ts`)
- **경로 별칭**: `@/` = 저장소 루트 (예: `@/lib/types`)

> 주의: `README.md`는 OpenAI라고 적혀 있고 `FOLDER_STRUCTURE.md`에는 아직 없는 파일이 많다. 실제 코드는 Gemini를 쓴다. 문서와 코드가 다르면 **코드를 기준**으로 한다.

### 폴더 역할

| 폴더/파일 | 역할 |
|---|---|
| `app/page.tsx` | 홈 — 일기 작성 화면 |
| `app/result/page.tsx` | 분석 결과 화면 |
| `app/history/page.tsx` | 기록/트렌드 화면 |
| `app/safety/page.tsx` | 위기 안내 화면 |
| `app/settings/page.tsx` | 설정 (백업/복원) |
| `app/api/analyze/route.ts` | `POST /api/analyze` — Gemini 호출, 모델 재시도, 실패 시 안전한 대체 응답 |
| `app/layout.tsx`, `app/globals.css` | 전역 레이아웃(한국어), Tailwind 진입점 |
| `components/` | 화면에서 쓰는 UI 컴포넌트 (현재 `BottomNav.tsx`) |
| `lib/types.ts` | 타입 정의 + zod 스키마 (감정 라벨, 액션, 분석 결과) |
| `lib/prompts.ts` | AI 시스템 프롬프트와 사용자 메시지 생성 |
| `lib/storage.ts` | localStorage 저장/불러오기 (CRUD) |
| `lib/validation.ts` | 일기 입력 검증 (20~2000자, 금칙어, 반복 문자) |
| `lib/emotions.ts` | 감정 라벨 → 이모지 매핑 |
| `lib/evidence.ts` | 추천 근거 설명 데이터 |
| `lib/catalogs/*.json` | 추천 카탈로그 (음악, 꽃, 활동, 호흡, 글쓰기) — AI는 여기 있는 항목만 추천해야 함 |
| `public/atti/` | 아띠 캐릭터 이미지 |
| `docs/` | 기획안, 아키텍처, 디자인 시스템, 진행 상황 |
| 루트 `*_RULES.md` 등 | `DESIGN_RULES.md`(디자인), `SAFETY_RULES.md`(안전/윤리), `ENV_SETUP.md`(환경 변수), `TEST_CHECKLIST.md`(테스트) |

## 2. 실행·빌드·린트 명령어 (`package.json` 기준)

```bash
npm install      # 패키지 설치 (처음 한 번)
npm run dev      # 개발 서버 실행 → http://localhost:3000
npm run build    # 프로덕션 빌드
npm run start    # 빌드된 앱 실행 (build 후)
npm run lint     # 코드 검사 (next lint)
```

- 테스트 스크립트는 아직 없다. 확인용 체크리스트는 `TEST_CHECKLIST.md`를 본다.
- 린트 설정 파일(`eslint.config.*`)이 저장소에 없어서 `npm run lint`가 기대대로 동작하지 않을 수 있다. 문제가 생기면 임의로 고치지 말고 먼저 알려준다.
- AI 분석을 로컬에서 쓰려면 루트에 `.env.local`을 만들고 `GOOGLE_API_KEY`를 넣는다 (`ENV_SETUP.md` 참고). `.env*`는 `.gitignore`로 제외되어 있다.

## 3. 작업 규칙 (사용자는 비전공자이고 학습 중이다)

1. **먼저 계획, 그다음 실행**: 코드를 바꾸기 전에 무엇을 어떻게 바꿀지 **한국어로 쉽게** 설명한다. 사용자가 **"진행해"**라고 말하면 그때 시작한다. 그 전에는 파일을 수정하지 않는다.
2. **바꾼 뒤 설명**: 작업 후 **무엇을, 왜 바꿨는지** 알려준다. 어려운 용어는 일상적인 **비유**와 함께 설명한다. (예: "API는 식당 주문 창구 같은 것")
3. **한 번에 한 가지만**: 한 번에 한 가지 작업만 한다. **요청하지 않은 파일은 건드리지 않는다.** 하고 싶은 다른 개선이 보이면 수정하지 말고 제안만 한다.
4. **브랜치와 커밋**: 항상 새 브랜치에서 작업한다. 커밋 메시지는 **한국어**로, **무엇을 왜 바꿨는지** 쓴다.
5. **민감 정보 보호**: API 키, `.env` 파일 내용, **사용자 일기 내용**은 코드·로그(`console.log` 포함)·커밋에 **절대** 넣지 않는다. 테스트용 예시 일기는 가짜 문장만 쓴다.
