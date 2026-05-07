# AI 교육 제안서 자동 생성기 — 임유진 모듈

> 교육 요청 정보를 입력하면 **Gemini(Google AI Studio)가 PPT 슬라이드 구성안을 자동 생성**합니다.  
> 이노핏파트너스 PPT 포맷 기반 — 과정 내용이 아닌 **슬라이드 구성 구조**를 템플릿으로 사용.

---

## 참조: PPT 슬라이드 구성 포맷

> 아래 구조를 그대로 재현하지 않고, 구성 패턴만 참고해 Gemini가 내용을 생성.

| 슬라이드 | 섹션 | 핵심 요소 |
|---------|------|----------|
| 1 | **표지** | 제안사명 / 날짜 / 과정명 |
| 2 | **과정 개요** | 목적(3가지 인사이트) / 대상·시간 / 교육방법(%) / 과정구성 / 기대효과(3가지) |
| 3~N | **모듈별 세부내용** | 모듈명 / 세부 설명 / 소요 시간 |
| N+1 | **추천 강사** | 강사 프로필·경력 |
| N+2 | **회사 소개** | 업체 정보·연혁·수행사례 |
| 마지막 | **고객 사례** | 레퍼런스 기업·성과 |

---

## 프론트엔드 계획

### 기술 스택

| 항목 | 선택 | 이유 |
|------|------|------|
| 프레임워크 | Next.js 14 (기존 템플릿) | 기존 프로젝트 활용 |
| AI | Gemini API (`@google/generative-ai`) | Google AI Studio 연동 |
| UI | shadcn/ui (Card·Badge·Tabs·DataTable) | KCH 공통 컴포넌트 |
| 저장 | Firestore `ai_proposals` | 이력 관리 |
| 스타일 | Tailwind + KCH 컬러(`#1962A8`/`#70AEDA`) | 공통 디자인 시스템 |

---

### 화면 구성 (3페이지)

#### Page 1 — `/ai-proposal` (인덱스)
```
┌─────────────────────────────────────────┐
│  KCH — AI 교육 제안서 생성기              │
│  요청 입력 → Gemini → PPT 구성안 자동 생성 │
├──────────────────┬──────────────────────┤
│  새 제안서 생성   │   제안서 이력 보기     │
│  [입력 폼으로 →] │   [저장 목록 →]       │
└──────────────────┴──────────────────────┘
```

#### Page 2 — `/ai-proposal/generate` (핵심 페이지)
```
┌──────────────────────┬──────────────────────────────┐
│   고객사 정보 입력    │    생성된 제안서 구조           │
│                      │                              │
│  고객사명: [    ]    │  ## 표지                      │
│  업종:    [    ]    │  ## 과정 개요                  │
│  기업규모: [▼  ]    │    - 목적 (3가지)              │
│                      │    - 대상/시간/방법            │
│  교육 요청:          │    - 기대효과 (3가지)          │
│  [             ]    │                              │
│  [             ]    │  ## 모듈 1: ___               │
│                      │    세부 내용...               │
│  교육 대상: [    ]  │                              │
│  희망 시간: [    ]  │  ## 모듈 2: ___               │
│                      │    세부 내용...               │
│  [제안서 생성 버튼]  │                              │
│                      │  ## 강사 프로필               │
│                      │  ## 기대 효과                 │
│                      │                              │
│                      │  [복사] [저장]               │
└──────────────────────┴──────────────────────────────┘
```

#### Page 3 — `/ai-proposal/history` (이력)
```
┌─────────────────────────────────────────────────┐
│  제안서 이력                          [새 생성]  │
├──────┬────────┬────────┬──────────┬─────────────┤
│ 날짜 │ 고객사 │  업종  │  상태   │    액션      │
├──────┼────────┼────────┼──────────┼─────────────┤
│ 5/7  │ ABC(주)│ 제조업 │ 초안    │ [보기][삭제] │
│ 5/6  │ DEF(주)│ 금융   │ 발송완료│ [보기]      │
└──────┴────────┴────────┴──────────┴─────────────┘
```

---

### 입력 필드 정의

| 필드 | 타입 | 필수 | 예시 |
|------|------|------|------|
| 고객사명 | text | ✅ | (주)ABC그룹 |
| 업종 | text | ✅ | 제조업 / 금융업 / 유통업 |
| 기업규모 | select | ✅ | 대기업(1000명↑) / 중견 / 중소 / 스타트업 |
| 교육 요청사항 | textarea | ✅ | "전사 AI 리터러시 향상, 실무 프롬프트 교육" |
| 교육 대상 | text | ✅ | 전사 임직원 200명 / IT 개발자 30명 |
| 희망 교육 시간 | text | - | 총 9시간(1일) / 3개월 12시간 |
| 예산 범위 | text | - | 1,000만원 이내 |

---

### Gemini 프롬프트 설계

**System Prompt 골격:**
```
당신은 기업 AI 교육 전문 컨설턴트입니다.
고객사 정보를 바탕으로 PPT 슬라이드 구성안을 작성합니다.

출력 구조 (반드시 준수):
1. 표지 정보 (과정명 / 날짜 / 제안사)
2. 과정 개요
   - 목적: Strategic / Leadership / Operational 관점 각 1줄
   - 대상 및 총 시간
   - 교육 방법 (이론 X% / 실습 X% / 워크숍 X%)
   - 기대효과 3가지
3. 모듈 구성 (2~4개)
   - 모듈명 / 소요 시간 / 세부 내용
4. 강사 프로필 개요 (1~2명 추천 기준)
5. 일정 제안
```

**User Prompt:** 고객사 입력값 → 위 구조에 맞게 내용 생성

---

### Gemini API 연동

```
모델: gemini-2.0-flash (속도·비용 균형)
패키지: @google/generative-ai
API 키: GEMINI_API_KEY (서버사이드 .env.local)
라우트: POST /api/proposal → 서버에서 Gemini 호출 후 결과 반환
```

---

### 파일 구조

```
src/
├── app/
│   ├── ai-proposal/
│   │   ├── page.tsx                   # 인덱스 (2-카드 그리드)
│   │   ├── generate/page.tsx          # 생성 페이지 (2-column)
│   │   └── history/page.tsx           # 이력 DataTable
│   └── api/
│       └── proposal/route.ts          # Gemini API 서버 라우트
├── lib/
│   ├── ai-proposal-domain.ts          # 슬라이드 구조 템플릿 + 프롬프트 빌더
│   └── ai-proposal-firestore.ts       # ai_proposals 컬렉션 어댑터
```

---

### Firestore 컬렉션

```
ai_proposals/
  {docId}/
    companyName:    string   # 고객사명
    industry:       string   # 업종
    companySize:    string   # 규모
    request:        string   # 요청사항
    targetAudience: string   # 교육 대상
    preferredHours: string   # 희망 시간
    result:         string   # Gemini 생성 결과 (마크다운)
    status:         'draft' | 'sent'
    _pushedAt:      Timestamp
```

---

### 환경 변수 (.env.local)

```bash
GEMINI_API_KEY=AIza...                    # Google AI Studio API 키
NEXT_PUBLIC_FIREBASE_API_KEY=...
NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN=...
NEXT_PUBLIC_FIREBASE_PROJECT_ID=...
NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET=...
NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID=...
NEXT_PUBLIC_FIREBASE_APP_ID=...
```

---

### 구현 순서

| # | 작업 | 예상 시간 |
|---|------|---------|
| 1 | `npm install @google/generative-ai` | 5분 |
| 2 | `src/lib/ai-proposal-domain.ts` — 슬라이드 템플릿 + 프롬프트 빌더 | 20분 |
| 3 | `src/app/api/proposal/route.ts` — Gemini API 서버 라우트 | 15분 |
| 4 | `src/lib/ai-proposal-firestore.ts` — Firestore 어댑터 | 10분 |
| 5 | `src/app/ai-proposal/page.tsx` — 인덱스 (2-카드) | 10분 |
| 6 | `src/app/ai-proposal/generate/page.tsx` — 핵심 생성 페이지 | 30분 |
| 7 | `src/app/ai-proposal/history/page.tsx` — DataTable 이력 | 20분 |

---

> 스택·보안·컬러 공통 룰: [`kch-hr-module-template/CLAUDE.md`](./kch-hr-module-template/CLAUDE.md) 참조.
