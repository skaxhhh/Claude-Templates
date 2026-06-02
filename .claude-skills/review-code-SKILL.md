---
name: review-code
auto-invoke: false
target-subagent: implementer
expose-in-menu: true
---

# 스킬: 코드 리뷰 및 품질 검증

## 설명

Implementer가 구현한 코드를 검증하고, 품질 기준을 충족하는지 확인합니다.
- TypeScript strict mode 준수
- 테스트 커버리지 확인
- 에러 처리 검증
- 성능 영향 평가
- 호환성 검증

## 사용 방법

```bash
# 기본 검토 (마지막 커밋)
/review-code

# 특정 파일만 검토
/review-code --files src/loaders/userLoader.ts

# 특정 범위 검토
/review-code --since last-tag

# 상세 검토 (모든 가이드라인 포함)
/review-code --detailed

# 빠른 검토 (핵심만)
/review-code --fast
```

## 검토 프로세스

### Step 1: 변경사항 파악 (2분)

```bash
# 최근 커밋 확인
git log --oneline -5

# 변경된 파일 확인
git diff --name-only HEAD~1..HEAD

# 변경 내용 확인
git diff HEAD~1..HEAD | head -100
```

### Step 2: TypeScript 검증 (3분)

```bash
# 타입 검사
npm run build

# ESLint 검사
npm run lint

# 결과 정리
# ✓ Build successful (또는 에러 목록)
# ✓ Linting passed (또는 경고 목록)
```

### Step 3: 테스트 검증 (5분)

```bash
# 해당 파일의 테스트 실행
npm test src/loaders/userLoader.test.ts

# 커버리지 확인
npm test -- --coverage

# 결과
# ✓ All tests passed: 312/312
# ✓ Coverage: 78% (target: 80%+)
```

### Step 4: 코드 품질 검토 (5분)

#### 체크리스트

- [ ] **TypeScript strict mode**
  ```typescript
  // ✅ 좋음
  const getUser = async (id: string): Promise<User> => { ... }
  
  // ❌ 나쁨
  const getUser = async (id) => { ... }
  ```

- [ ] **명확한 타입 지정**
  ```typescript
  // ❌ 나쁨
  const result: any = fetchData();
  
  // ✅ 좋음
  const result: User | null = fetchData();
  ```

- [ ] **에러 처리**
  ```typescript
  // ✅ 좋음
  try {
    const user = await User.findById(id);
    return user;
  } catch (error) {
    logger.error('Failed to fetch user', { error, userId: id });
    throw new AppError('User not found', 404);
  }
  
  // ❌ 나쁨
  const user = await User.findById(id);  // 에러 처리 없음
  return user;
  ```

- [ ] **복잡한 로직에 주석**
  ```typescript
  // ✅ 좋음: 복잡한 로직에는 주석
  // DataLoader는 같은 이벤트 루프에서 호출된 여러 load()를
  // 배치 쿼리로 변환하여 N+1 문제 해결
  const userLoader = new DataLoader(async (userIds) => {
    return User.findByIds(userIds);
  });
  
  // ❌ 나쁨: 간단한 로직에 과도한 주석
  // Set i to 0
  let i = 0;
  ```

- [ ] **네이밍 규칙**
  ```typescript
  // ✅ 좋음
  const getUserByEmail = () => {}      // 함수: camelCase
  const MAX_RETRIES = 3                // 상수: UPPER_SNAKE_CASE
  class UserService { }                // 클래스: PascalCase
  
  // ❌ 나쁨
  const get_user_by_email = () => {}   // 스네이크 케이스
  const max_retries = 3                // 스네이크 케이스
  ```

- [ ] **불필요한 코드 없음**
  ```typescript
  // ✅ 정리됨
  import { User } from '../models/User';
  import { logger } from '../utils/logger';
  
  // ❌ 불필요한 import
  import { unused } from '../utils/unused';
  import type { UnusedType } from '../types';
  ```

### Step 5: 성능 영향 평가 (3분)

- [ ] **새로운 N+1 쿼리 도입 없음**
  ```typescript
  // ❌ 나쁨: N+1 쿼리
  const posts = await Post.find({ userId });
  const users = posts.map(post => User.findById(post.userId));
  
  // ✅ 좋음: DataLoader 또는 JOIN
  const posts = await Post.find({ userId }).populate('user');
  ```

- [ ] **불필요한 데이터 로드 없음**
  ```typescript
  // ❌ 나쁨: 모든 필드를 로드
  const user = await User.findById(id);
  
  // ✅ 좋음: 필요한 필드만
  const user = await User.findById(id, { name: 1, email: 1 });
  ```

- [ ] **메모리 누수 없음**
  ```typescript
  // ❌ 나쁨: 구독 해제 없음
  this.stream.on('data', (chunk) => { ... });
  
  // ✅ 좋음: 정리됨
  const listener = (chunk) => { ... };
  this.stream.on('data', listener);
  
  ngOnDestroy() {
    this.stream.removeListener('data', listener);
  }
  ```

### Step 6: 호환성 검증 (2분)

- [ ] **Breaking changes 없음** (설계에 없으면)
  ```typescript
  // ❌ 나쁨: API 변경
  - app.get('/api/users/:id', handler);
  
  // ✅ 좋음: 새 버전 추가, 기존 유지
  app.get('/api/users/:id', handler);      // 기존
  app.get('/api/v2/users/:id', handler);   // 신규
  ```

- [ ] **Deprecation 기간 제공** (있으면)
  ```typescript
  /**
   * @deprecated Use getActiveUsers() instead (2026-09-01 제거 예정)
   */
  const getAllUsers = () => { ... }
  ```

- [ ] **테스트 호환성**
  ```bash
  # 기존 테스트 모두 통과
  npm test
  # ✓ 312/312 passed
  ```

## 산출물 형식

검토 완료 후 다음 형식으로 결과 반환:

```json
{
  "review_summary": {
    "status": "✓ APPROVED",
    "total_issues": 0,
    "critical_issues": 0,
    "warnings": 0
  },
  "typescript_check": {
    "build_status": "✓ SUCCESS",
    "strict_mode": "✓ ENABLED",
    "errors": 0,
    "warnings": 0
  },
  "linting_check": {
    "status": "✓ PASSED",
    "errors": 0,
    "warnings": 0
  },
  "test_check": {
    "unit_tests": "✓ 312/312 PASSED",
    "test_coverage": "78%",
    "new_tests": 12,
    "status": "✓ PASSED"
  },
  "code_quality": {
    "items_checked": [
      {
        "category": "TypeScript Strict Mode",
        "status": "✓",
        "details": "All functions have explicit return types"
      },
      {
        "category": "Error Handling",
        "status": "✓",
        "details": "All async operations wrapped in try-catch"
      },
      {
        "category": "Comments & Documentation",
        "status": "✓",
        "details": "Complex logic documented (DataLoader batching)"
      },
      {
        "category": "Naming Conventions",
        "status": "✓",
        "details": "camelCase functions, UPPER_SNAKE_CASE constants"
      },
      {
        "category": "Dead Code",
        "status": "✓",
        "details": "No unused imports or variables"
      }
    ]
  },
  "performance_impact": {
    "n_plus_one_queries": "✓ None introduced",
    "memory_leaks": "✓ None detected",
    "data_loading": "✓ Optimized (uses DataLoader)",
    "overall_impact": "✓ Positive"
  },
  "compatibility": {
    "breaking_changes": "✓ None",
    "api_changes": "✓ None (backward compatible)",
    "test_compatibility": "✓ All existing tests pass"
  },
  "commit_quality": {
    "message_quality": "✓ Clear and descriptive",
    "commit_size": "✓ Appropriate",
    "related_changes": "✓ Well grouped"
  },
  "final_verdict": {
    "status": "✓ APPROVED",
    "can_merge": true,
    "ready_for_testing": true,
    "comments": "Well-implemented DataLoader with comprehensive tests. No issues found."
  },
  "reviewer": "review-code Skill",
  "timestamp": "2026-06-02T15:30:00Z"
}
```

## 검토 체크리스트

### Pre-Merge
- [ ] ✓ TypeScript strict mode 통과
- [ ] ✓ ESLint 통과 (에러 0개)
- [ ] ✓ 모든 테스트 통과 (312/312)
- [ ] ✓ 테스트 커버리지 ≥ 80%
- [ ] ✓ 복잡한 로직 주석 처리
- [ ] ✓ Breaking change 없음
- [ ] ✓ 에러 처리 완벽

### Performance
- [ ] ✓ 새로운 N+1 쿼리 없음
- [ ] ✓ 불필요한 데이터 로드 없음
- [ ] ✓ 메모리 누수 없음

### Code Quality
- [ ] ✓ 네이밍 규칙 준수
- [ ] ✓ 불필요한 코드 없음
- [ ] ✓ Import 정렬됨

## 실패 시 절차

검토에서 실패한 항목이 있으면:

1. **Issue 명시**
   - 파일명과 라인 번호
   - 문제점 설명
   - 해결 방법 제시

2. **Implementer 수정**
   - 지적된 항목 수정
   - 다시 `/review-code` 실행

3. **재검토**
   - 수정 사항 확인
   - 모든 항목 통과할 때까지 반복

## 참고사항

- 이 스킬은 Implementer가 커밋 전에 자동 호출
- 불명확한 항목은 무조건 flags (완벽함 > 빠름)
- "좋은 코드 같은데..." 느낌의 항목도 체크
- 최종 승인은 Tester가 전체 테스트 후 진행
