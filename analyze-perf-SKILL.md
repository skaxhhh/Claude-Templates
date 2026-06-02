---
name: analyze-perf
auto-invoke: false
target-subagent: analyzer
expose-in-menu: true
---

# 스킬: 성능 분석

## 설명

코드베이스의 성능 문제를 체계적으로 분석합니다.
- N+1 쿼리 패턴 식별
- API 응답시간 병목 찾기
- 테스트 커버리지 갭 식별
- 아키텍처 개선 기회 제시

## 사용 방법

```bash
# 전체 분석
/analyze-perf

# 특정 영역만 분석
/analyze-perf --scope database      # DB 쿼리만
/analyze-perf --scope api           # API 응답시간만
/analyze-perf --scope coverage      # 테스트 커버리지만
/analyze-perf --scope architecture  # 아키텍처만

# 상세 분석
/analyze-perf --detailed
```

## 분석 프로세스

### 1단계: 저장소 구조 파악

```bash
# 파일 수 확인
find . -name "*.ts" -o -name "*.js" | wc -l

# 코드 라인 수 확인
find src -name "*.ts" | xargs wc -l | tail -1

# 의존성 구조 파악
grep -r "^import\|^require" --include="*.ts" --include="*.js" src/ | head -30
```

### 2단계: 데이터베이스 성능 분석

```bash
# N+1 쿼리 패턴 찾기 (몇 가지 패턴)
# 패턴 1: for 루프 내 DB 호출
grep -n "for.*in\|forEach" src/**/*.ts | grep -A5 "findOne\|find\|query"

# 패턴 2: 맵 내 DB 호출
grep -n "\.map(" src/**/*.ts | grep -A2 "await"

# 패턴 3: Promise.all 없는 순차 호출
grep -B5 -A5 "await.*await" src/**/*.ts | head -30
```

### 3단계: API 응답시간 분석

```bash
# 느린 엔드포인트 찾기
npm test -- --coverage 2>&1 | grep "slow\|timeout"

# 주요 엔드포인트 응답시간 측정
ab -n 100 -c 10 http://localhost:3000/api/users
# 결과: Requests per second, Time per request 확인
```

### 4단계: 테스트 커버리지 분석

```bash
# 전체 커버리지 확인
npm test -- --coverage

# 커버리지 리포트 분석
cat coverage/coverage-summary.json | jq '.total'
```

### 5단계: 아키텍처 분석

```bash
# 파일 간 의존성 검사
# 순환 의존성 찾기
grep -r "^import.*from\|^require" src/ | \
  awk '{print $3}' | sort | uniq -d

# 모놀리식 구조 식별
ls -la src/services/ | wc -l  # 서비스 수
ls -la src/models/ | wc -l    # 모델 수
```

## 산출물 형식

분석 완료 후 다음 형식으로 결과 반환:

```json
{
  "analysis_summary": {
    "total_files": 127,
    "total_lines": 45000,
    "languages": ["TypeScript", "SQL"],
    "frameworks": ["Express", "TypeORM"]
  },
  "database_analysis": {
    "n_plus_one_queries": 12,
    "critical_queries": [
      {
        "location": "src/api/users.ts:45",
        "pattern": "User.findMany() → Post.find() per user",
        "severity": "high",
        "queries_produced": "1 + N (N = 사용자 수)",
        "fix_effort_hours": 2,
        "fix_method": "DataLoader 도입"
      }
    ],
    "missing_indexes": 3,
    "slow_queries": []
  },
  "api_analysis": {
    "total_endpoints": 28,
    "slow_endpoints": [
      {
        "path": "GET /api/users",
        "avg_response_ms": 850,
        "p95_response_ms": 1200,
        "bottleneck": "N+1 queries on related posts"
      }
    ]
  },
  "testing_analysis": {
    "current_coverage": "67%",
    "target_coverage": "95%",
    "coverage_gap": "28%",
    "untested_critical_paths": 12,
    "priority_files": [
      "src/api/auth.ts",
      "src/services/payment.ts",
      "src/services/email.ts"
    ]
  },
  "architecture_analysis": {
    "structure": "monolithic",
    "concerns": [
      "Auth logic mixed with business logic",
      "Shared database, no service boundaries"
    ],
    "microservice_candidates": [
      {
        "service_name": "Auth Service",
        "files": ["src/services/auth.ts", "src/models/User.ts"],
        "dependencies": 3,
        "complexity": "medium"
      }
    ]
  },
  "top_5_recommendations": [
    {
      "rank": 1,
      "title": "DataLoader 도입",
      "effort_hours": 2,
      "impact": "N+1 제거, 응답시간 30% 개선",
      "priority": "critical"
    },
    {
      "rank": 2,
      "title": "DB 인덱스 추가",
      "effort_hours": 1,
      "impact": "쿼리 성능 20% 개선",
      "priority": "high"
    },
    {
      "rank": 3,
      "title": "테스트 추가 (Auth)",
      "effort_hours": 2,
      "impact": "커버리지 67% → 75%",
      "priority": "high"
    },
    {
      "rank": 4,
      "title": "Auth Service 분리",
      "effort_hours": 3,
      "impact": "아키텍처 개선, 확장성 증대",
      "priority": "medium"
    },
    {
      "rank": 5,
      "title": "캐싱 레이어 추가",
      "effort_hours": 2,
      "impact": "응답시간 50% 추가 개선",
      "priority": "medium"
    }
  ]
}
```

## 다음 단계

분석 완료 후:
1. **Designer**가 이 결과를 바탕으로 구체적인 개선 계획 수립
2. **Implementer**가 계획을 코드로 구현
3. **Tester**가 개선 효과 검증

## 참고사항

- 분석은 읽기 전용 (코드 수정 금지)
- 기존 테스트는 실행하지 않음 (별도 스킬 또는 Tester)
- 분석 결과는 JSON 형식 (다음 Agent가 자동 파싱 가능)
