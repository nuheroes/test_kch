# mjs-leaves — 이재성 M2 연차 자동 산출

> KCH그룹 인사총무팀 14h AI 워크숍 — **이재성 개인 repo**.
> 본 repo는 [`kch-hr-module-template`](../kch-hr-module-template/)에서 fork됩니다. 공통 자산(스택·디자인·보안)은 그대로 두고, 본인 모듈만 채웁니다.

---

## 1줄 요약

> 법정 + 근속 + 해외 자동 산정 → 분기 사업부장 메일 (Cloud Scheduler `0 9 1 1,4,7,10 *`).

**핵심 함수**: `getLeaves(empId): LeaveBalance` — 직원 ID 1개로 모든 휴가 잔여 한 번에 반환.

---

## 본인 모듈 폴더 구조

```
src/
├── app/
│   └── m2-leaves/                     ← 본인 모듈 (D1 #4 cp -r)
│       ├── page.tsx                   # 홈 — 사업부 4개 카드
│       ├── dept/[id]/page.tsx         # 사업부별 집계
│       ├── employee/[id]/page.tsx     # 개인별 잔여 (핵심 페이지)
│       ├── quarterly/page.tsx         # 분기 리포트
│       └── erp-upload/page.tsx        # ERP CSV 붙여넣기
├── lib/
│   ├── m2-domain.ts                   # 산정 룰 (법정·근속·해외)
│   └── m2-firestore.ts                # Firestore 어댑터
└── ...

functions/src/
├── index.ts
└── m2-automation.ts                   ← 분기 메일 (D2 #7)
```

---

## Firestore 컬렉션 (4개)

| 컬렉션 | 의미 | 핵심 필드 |
|---|---|---|
| `employees` | 직원 마스터 | `empId(token)`, `joinDate`, `isOverseas`, `deptId`, `salaryToken` |
| `leave_balances` | 휴가 잔여 (연도별 1행) | `empId`, `fiscalYear`, `legal`, `seniority`, `special`, `overseas`, `total` |
| `leave_usages` | 휴가 사용 트랜잭션 | `empId`, `usedDate`, `category`, `days`, `reason` |
| `org_units` | 조직 마스터 | `deptId`, `deptName`, `headEmail` |

---

## 자동화 게이트 — 분기 메일

### 함수: `quarterlyMail`

- **트리거**: Cloud Scheduler `0 9 1 1,4,7,10 *` (분기 첫날 09:00 KST)
- **동작**: 4개 사업부에 대해 분기 휴가 사용 현황을 집계 → Gmail Draft 4건 자동 생성
- **자동 send X**: Draft만 생성. 사람이 검수 후 발송 결정 (분기 메일 사고 방지)

### 검증 게이트

- 4개 사업부 모두 Draft 생성됨
- 각 Draft에 사업부별 데이터 포함 (다른 사업부 데이터 누수 0건)
- 직원 식별 컬럼 평문 0건 (token만)

---

## 사전 확정 4건 (D1 시작 전)

본 repo를 clone하기 전에 다음 4건이 결정되어 있어야 합니다. [사전준비-체크리스트.md](../../교안/사전준비-체크리스트.md)를 참고하세요.

1. **회계년도 기준일** — 1/1 vs 4/1 vs 입사월
2. **ERP 추출 컬럼 구조** — CSV 샘플 1건 (인사정보 토큰 치환 후)
3. **해외 14일 사용 후 잔여 처리** — 이월 / 소멸 / 환산
4. **분기 메일 수신자/시점/포맷** — 본인 + 팀장 + CFO? / 분기 1일? / HTML 표

---

## 슬래시커맨드 1개

`/seed-erp` — ERP CSV 더미 1건 → Firestore 변환.

`.claude/commands/seed-erp.md` 파일에 다음 내용을 적습니다 (D2 #6).

---

## 30/60/90일 자력 로드맵

- **30일**: ERP 직접 연동 (CSV import → API 호출). 분기 메일 자동 send (Draft → Send).
- **60일**: 채용 평가 자동화 (M2 코드 80% 재사용). 복지포인트·생일 쿠폰. 연차수당 자동 계산.
- **90일**: Apps Script 자산 → Cloud Functions 마이그레이션. 이윤재 SaaS와 데이터 통합.

---

## 본인 URL 기록

```
본인 URL: ________________________________________________
```

D2 #8 통합 시점에 이 URL을 이윤재(M4)에게 전달합니다.

---

> 본 README의 상세 셋업은 [`교안/셋업-이재성-M2연차.md`](../../교안/셋업-이재성-M2연차.md) 참조.
