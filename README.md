# 프로젝트별 Subagent 템플릿 사용 가이드

이 디렉토리에는 모든 프로젝트에서 사용할 수 있는 **범용 템플릿**이 포함되어 있습니다.

---

## 📁 디렉토리 구조

```
templates/
├── .claude-agents/          # Agent 정의 템플릿
│   ├── analyzer.md          # 분석 전문가
│   ├── designer.md          # 아키텍처 설계자
│   ├── implementer.md       # 수석 엔지니어
│   └── tester.md            # QA 엔지니어
│
├── .claude-skills/          # Skill 매크로 템플릿
│   ├── analyze-perf-SKILL.md        # 성능 분석 스킬
│   ├── design-service-SKILL.md      # 서비스 설계 스킬
│   └── review-code-SKILL.md         # 코드 리뷰 스킬
│
├── .claude-hooks/           # 자동화 규칙 템플릿
│   └── hooks.yaml           # 자동 검증/포매팅
│
└── README.md               # 이 파일
```

---

## 🚀 빠른 시작 (5분)

### Step 1: 프로젝트에 템플릿 복사

```bash
cd ~/your-project

# 템플릿 디렉토리 생성
mkdir -p .claude/agents .claude/skills/{analyze-perf,design-service,review-code} .claude/hooks

# 이 템플릿들을 복사
cp templates/.claude-agents/*.md .claude/agents/
cp templates/.claude-skills/*.md .claude/skills/analyze-perf/SKILL.md
cp templates/.claude-skills/*.md .claude/skills/design-service/SKILL.md
cp templates/.claude-skills/*.md .claude/skills/review-code/SKILL.md
cp templates/.claude-hooks/hooks.yaml .claude/hooks/
```

### Step 2: 프로젝트별로 커스터마이징

각 파일에서:
1. `[프로젝트명]` 텍스트를 실제 프로젝트명으로 변경
2. 해당되지 않는 섹션 제거
3. 프로젝트 특화 내용 추가

### Step 3: 바로 사용

```bash
# Orchestrator에게 명령
당신: "프로젝트 성능 개선해줘. 분석부터 시작해"

# 또는 특정 Agent 호출
당신: "/analyze-perf"
```

---

## 📊 각 파일 설명

### Agents (.claude/agents/)

#### 1. **analyzer.md** - 분석 전문가
```
역할: 코드와 성능 현황 파악
입력: 프로젝트 요구사항
출력: JSON 형식 분석 결과
```

**분석 항목:**
- N+1 쿼리 발견
- API 응답시간 병목
- 테스트 커버리지 갭
- 아키텍처 개선 기회

**예시:**
```bash
# Orchestrator가 자동으로 호출하거나 수동 호출:
/analyze-perf --scope database
```

#### 2. **designer.md** - 아키텍처 설계자
```
역할: 구체적인 개선 계획 수립
입력: Analyzer의 분석 결과
출력: Markdown 상세 설계 문서 (Phase 1/2/3)
```

**설계 항목:**
- 페이즈별 작업 목록
- 예상 시간 및 난이도
- 위험도 평가
- 롤백 전략

**예시:**
```bash
# Analyzer 다음에 자동 호출
당신: "설계해줘"
# → Designer가 3단계 계획 수립
```

#### 3. **implementer.md** - 수석 엔지니어
```
역할: 코드 구현
입력: Designer의 설계 계획
출력: Git commits (구현된 코드)
```

**구현 항목:**
- 각 Phase별 코드 작성
- 테스트 작성
- 커밋 메시지 작성

**체크리스트:**
- ✓ 모든 테스트 통과
- ✓ Linting 통과
- ✓ TypeScript 컴파일 성공

**예시:**
```bash
# Designer 다음에 자동 호출
당신: "구현해줘"
# → Implementer가 코드 작성 + 테스트 + 커밋
```

#### 4. **tester.md** - QA 엔지니어
```
역할: 품질 보증 및 검증
입력: Implementer의 구현 결과
출력: JSON 형식 QA 리포트
```

**검증 항목:**
- 모든 테스트 통과 확인
- 성능 개선 측정 (Before/After)
- 회귀 테스트
- 보안 검증

**예시:**
```bash
# Implementer 다음에 자동 호출
당신: "검증해줘"
# → Tester가 품질 검증 + 성능 측정 + 승인
```

---

### Skills (.claude/skills/)

#### 1. **analyze-perf-SKILL.md**
```
목적: 성능 분석 자동화
호출: /analyze-perf [--scope database|api|coverage]
Agent: Analyzer가 사용
```

**분석 범위:**
- 데이터베이스 (N+1 쿼리, 느린 쿼리)
- API (응답시간, 병목)
- 테스트 (커버리지 갭)
- 아키텍처 (마이크로서비스 후보)

#### 2. **design-service-SKILL.md**
```
목적: 아키텍처 설계 자동화
호출: /design-service [--detailed]
Agent: Designer가 사용
```

**산출물:**
- Phase 1/2/3 상세 계획
- 각 Phase의 작업 목록
- 시간/노력 추정
- 위험도 및 롤백 계획

#### 3. **review-code-SKILL.md**
```
목적: 코드 품질 자동 검토
호출: /review-code [--detailed]
Agent: Implementer가 자신의 코드를 검토할 때
```

**검증 항목:**
- TypeScript strict mode
- 테스트 커버리지
- 에러 처리
- 성능 영향 평가
- 호환성 검증

---

### Hooks (.claude/hooks/hooks.yaml)

```
목적: 자동 검증 및 포매팅
작동: 파일 저장 시, 커밋 시, bash 명령 실행 시
```

**포함된 Hook:**
1. **auto-format-typescript** - Prettier 자동 실행
2. **block-debug-code** - console.log 커밋 방지
3. **block-secrets** - 비밀 정보 커밋 방지
4. **test-gate** - 커밋 전 테스트 실행 (선택)
5. **validate-json-files** - JSON 유효성 검증
6. **validate-migrations** - DB 마이그레이션 검증
7. **auto-compile-ts** - TypeScript 컴파일 체크 (선택)
8. **log-work** - 작업 내역 기록 (선택)

---

## 🔄 전체 워크플로우

```
당신의 요구사항
    ↓
Orchestrator가 이해하고 필요한 Agent 호출
    ↓
┌─────────────────────────────────────────────────────────┐
│                                                         │
│  [1] Analyzer 실행 → /analyze-perf skill              │
│      (분석 결과: JSON)                                  │
│           ↓                                             │
│  [2] Designer 실행 → /design-service skill            │
│      (설계 문서: 3단계 계획)                             │
│           ↓                                             │
│  [3] Implementer 실행 → /review-code skill (자가검사)   │
│      (구현 결과: Git commits)                           │
│           ↓                                             │
│  [4] Tester 실행 → QA 리포트 (JSON)                    │
│      (검증 결과: 성능 측정, 회귀 테스트)                 │
│           ↓                                             │
│  [5] Orchestrator 종합 → 최종 리포트                    │
│                                                         │
└─────────────────────────────────────────────────────────┘
    ↓
완료! 배포 준비 완료
```

---

## 🎯 각 프로젝트별 적용 방법

### 새 프로젝트 시작

```bash
# 1. 프로젝트 생성
mkdir my-project
cd my-project

# 2. 템플릿 복사 및 구조 생성
mkdir -p .claude/agents .claude/skills .claude/hooks

# 3. 각 템플릿 파일 복사
# 이때 프로젝트명을 변경

# 4. hooks.yaml 설정 (선택)
# 프로젝트별로 필요한 hook만 활성화

# 5. 사용 시작
당신: "프로젝트 성능을 개선해줘"
```

### 기존 프로젝트에 추가

```bash
# 현재 프로젝트에 템플릿 적용
cp -r templates/.claude-agents .claude/
cp -r templates/.claude-skills .claude/
cp -r templates/.claude-hooks .claude/

# 기존 코드와 충돌 확인
git status

# 프로젝트 특화 내용으로 커스터마이징
# (각 .md 파일 편집)

# 변경사항 커밋
git add .claude/
git commit -m "setup: Add Claude Code subagent configuration"
```

---

## ⚙️ 프로젝트별 커스터마이징 체크리스트

### Agent 커스터마이징

- [ ] **analyzer.md**
  - 프로젝트 특화 분석 항목 추가 (필요시)
  - 예: Redis 캐시 분석, 마이크로서비스 경계 추가

- [ ] **designer.md**
  - Phase 개수 조정 (2개, 3개, 4개?)
  - 예상 시간 프로젝트에 맞게 조정

- [ ] **implementer.md**
  - 코딩 스타일 가이드 추가
  - 프로젝트 특화 패턴 명시

- [ ] **tester.md**
  - 성능 목표 수치 명시
  - 추가 테스트 항목 (보안, 성능 등)

### Skill 커스터마이징

- [ ] **analyze-perf-SKILL.md**
  - 프로젝트 특화 분석 scope 추가
  - 예: Redis, Elasticsearch 분석

- [ ] **design-service-SKILL.md**
  - 프로젝트 특화 배포 전략
  - 예: Kubernetes, Docker 포함

- [ ] **review-code-SKILL.md**
  - 프로젝트별 코드 스타일
  - 예: 특정 라이브러리 사용 가이드

### Hooks 커스터마이징

- [ ] **hooks.yaml**
  - 불필요한 hook 비활성화 (disabled: true)
  - 프로젝트 특화 hook 추가
  - Context 지정 (analyzer, designer 등)

---

## 📝 사용 예시

### 예시 1: 성능 최적화 프로젝트

```bash
당신: "API 응답시간 50% 단축, 테스트 95% 커버리지"

1단계 - Analyzer (/analyze-perf)
→ N+1 쿼리 12개, 테스트 갭 28개 발견

2단계 - Designer (/design-service)
→ Phase 1 (Quick wins): DataLoader, 인덱스, 테스트 추가
→ Phase 2 (Architecture): Auth 분리, 카나리 배포
→ Phase 3 (Optimization): 캐싱, 쿼리 최적화

3단계 - Implementer (/review-code)
→ 코드 작성 + 테스트 + 커밋

4단계 - Tester
→ 응답시간: 850ms → 400ms (53% 개선) ✓
→ DB 쿼리: 45 → 8개 (82% 감소) ✓
→ 테스트: 95% 커버리지 ✓

결과: 모든 목표 달성!
```

### 예시 2: 마이크로서비스 마이그레이션

```bash
당신: "모놀리식을 마이크로서비스로 마이그레이션"

Analyzer: 아키텍처 분석 → Auth, API, Worker 경계 제안

Designer: 3단계 마이그레이션 계획
  Phase 1: 기초 (DataLoader, 테스트 추가)
  Phase 2: Auth 분리 (카나리 배포)
  Phase 3: Worker 분리 (비동기 작업)

Implementer: 각 Phase 구현

Tester: 각 Phase 검증
```

### 예시 3: 긴급 버그 수정

```bash
당신: "로그인 버그 수정"

Analyzer: 불필요 (간단한 버그)
Designer: 불필요 (설계 불필요)

→ 직접 Implementer 호출
당신: "로그인 버그 수정해줘"

Implementer: 코드 수정 + 테스트

Tester: 회귀 테스트 + 승인
```

---

## 🔧 트러블슈팅

### Q: Analyzer를 또 실행하고 싶은데?
```bash
당신: "/analyze-perf --scope api"
# 또는
당신: "/analyze-perf --detailed"
```

### Q: Designer의 설계를 수정하고 싶은데?
```bash
당신: "Phase 2의 예상 시간을 3일로 줄여줄 수 있어?"
# Designer가 문서를 업데이트함
```

### Q: Implementer가 특정 파일만 수정하고 싶은데?
```bash
당신: "src/api/users.ts만 리팩토링해"
# Implementer가 해당 파일만 수정
```

### Q: 특정 Hook을 비활성화하고 싶은데?
```yaml
# .claude/hooks/hooks.yaml에서
- name: test-gate
  disabled: true  # 이렇게 변경
```

---

## 📚 추가 리소스

- **Analyzer 상세 가이드**: .claude-agents/analyzer.md
- **Designer 상세 가이드**: .claude-agents/designer.md
- **Implementer 상세 가이드**: .claude-agents/implementer.md
- **Tester 상세 가이드**: .claude-agents/tester.md
- **Hooks 커스터마이징**: .claude-hooks/hooks.yaml

---

## ✅ 체크리스트: 프로젝트 준비 완료

프로젝트에 Subagent를 적용하기 전에:

- [ ] `.claude/agents/` 디렉토리 생성
- [ ] 4개 agent 파일 복사 (analyzer, designer, implementer, tester)
- [ ] `.claude/skills/` 디렉토리 생성
- [ ] 3개 skill 파일 복사 (analyze-perf, design-service, review-code)
- [ ] `.claude/hooks/` 디렉토리 생성 (선택)
- [ ] hooks.yaml 복사 (선택)
- [ ] 각 파일에서 `[프로젝트명]` 텍스트 변경
- [ ] 불필요한 섹션 제거
- [ ] 프로젝트 특화 내용 추가
- [ ] Git에 커밋

---

**모든 준비가 완료되었습니다!**

이제 Orchestrator에게 요구사항을 전달하면:
1. Analyzer가 분석
2. Designer가 계획 수립
3. Implementer가 구현
4. Tester가 검증

완전 자동화된 워크플로우를 즐기세요! 🚀
