# 🎯 TypeScript 모범 사례 가이드

## 📋 개요

이 문서는 GitHub Actions Demo 프로젝트에서 TypeScript를 효과적으로 사용하기 위한 모범 사례를 설명합니다.

## 🏗️ 프로젝트 구조 모범 사례

### 1. 폴더 구조
```
src/
├── types/           # 타입 정의
├── interfaces/      # 인터페이스 정의
├── enums/          # 열거형 정의
├── constants/      # 상수 정의
├── utils/          # 유틸리티 함수
├── services/       # 비즈니스 로직
├── controllers/    # 컨트롤러
├── middleware/     # 미들웨어
├── routes/         # 라우트 정의
├── config/         # 설정 파일
├── models/         # 데이터 모델
└── app.ts          # 메인 애플리케이션
```

### 2. 파일 명명 규칙
- **타입 정의**: `*.types.ts` 또는 `*.interface.ts`
- **상수**: `*.constants.ts`
- **유틸리티**: `*.utils.ts`
- **서비스**: `*.service.ts`
- **컨트롤러**: `*.controller.ts`

## 🔧 타입 정의 모범 사례

### 1. 기본 타입 정의
```typescript
// ✅ 좋은 예: 명확하고 구체적인 타입
interface User {
  readonly id: string;
  username: string;
  email: string;
  createdAt: Date;
  updatedAt: Date;
  isActive: boolean;
}

// ❌ 나쁜 예: 너무 일반적인 타입
interface User {
  id: any;
  data: any;
}
```

### 2. 유니온 타입 활용
```typescript
// ✅ 좋은 예: 명확한 상태 정의
type UserStatus = 'active' | 'inactive' | 'pending' | 'suspended';

interface User {
  id: string;
  status: UserStatus;
}

// ❌ 나쁜 예: 문자열로 상태 관리
interface User {
  id: string;
  status: string; // 어떤 값이 올 수 있는지 불명확
}
```

### 3. 제네릭 활용
```typescript
// ✅ 좋은 예: 재사용 가능한 제네릭 타입
interface ApiResponse<T> {
  success: boolean;
  data: T;
  error?: string;
  timestamp: string;
}

// 사용 예시
type UserResponse = ApiResponse<User>;
type UsersResponse = ApiResponse<User[]>;
```

### 4. 옵셔널 속성과 기본값
```typescript
// ✅ 좋은 예: 옵셔널 속성과 기본값 활용
interface CreateUserRequest {
  username: string;
  email: string;
  firstName?: string;
  lastName?: string;
  isActive?: boolean;
}

class UserService {
  createUser(data: CreateUserRequest): User {
    return {
      id: generateId(),
      username: data.username,
      email: data.email,
      firstName: data.firstName ?? '',
      lastName: data.lastName ?? '',
      isActive: data.isActive ?? true,
      createdAt: new Date(),
      updatedAt: new Date()
    };
  }
}
```

## 🛡️ 타입 안전성 모범 사례

### 1. 타입 가드 활용
```typescript
// ✅ 좋은 예: 타입 가드 함수
function isUser(obj: any): obj is User {
  return (
    typeof obj === 'object' &&
    obj !== null &&
    typeof obj.id === 'string' &&
    typeof obj.username === 'string' &&
    typeof obj.email === 'string' &&
    obj.createdAt instanceof Date &&
    obj.updatedAt instanceof Date &&
    typeof obj.isActive === 'boolean'
  );
}

// 사용 예시
function processUser(data: unknown): User | null {
  if (isUser(data)) {
    // 여기서 data는 User 타입으로 추론됨
    return data;
  }
  return null;
}
```

### 2. never 타입 활용
```typescript
// ✅ 좋은 예: never 타입으로 완전성 보장
type UserStatus = 'active' | 'inactive' | 'pending';

function getUserStatusColor(status: UserStatus): string {
  switch (status) {
    case 'active':
      return 'green';
    case 'inactive':
      return 'red';
    case 'pending':
      return 'yellow';
    default:
      // 모든 케이스를 처리했음을 보장
      const _exhaustiveCheck: never = status;
      throw new Error(`Unhandled status: ${_exhaustiveCheck}`);
  }
}
```

### 3. 타입 단언 최소화
```typescript
// ❌ 나쁜 예: 과도한 타입 단언
const user = data as User; // 위험함

// ✅ 좋은 예: 타입 가드 사용
const user = isUser(data) ? data : null;

// ✅ 좋은 예: 타입 단언이 필요한 경우 명확한 이유 제공
const user = data as User; // API 응답이 항상 User 타입임을 보장
```

## 🎨 코드 스타일 모범 사례

### 1. 인터페이스 vs 타입 별칭
```typescript
// ✅ 인터페이스: 객체 구조 정의에 사용
interface User {
  id: string;
  username: string;
}

// ✅ 타입 별칭: 유니온, 교집합, 기본 타입에 사용
type UserStatus = 'active' | 'inactive';
type UserWithStatus = User & { status: UserStatus };
```

### 2. 읽기 전용 속성 활용
```typescript
// ✅ 좋은 예: 불변성 보장
interface User {
  readonly id: string;
  readonly createdAt: Date;
  username: string;
  email: string;
}

// ✅ 좋은 예: Readonly 유틸리티 타입 활용
type ReadonlyUser = Readonly<User>;
```

### 3. 함수 타입 정의
```typescript
// ✅ 좋은 예: 명확한 함수 타입 정의
type EventHandler<T> = (event: T) => void;
type AsyncHandler<T, R> = (data: T) => Promise<R>;

// 사용 예시
const userCreatedHandler: EventHandler<User> = (user) => {
  console.log(`User created: ${user.username}`);
};

const fetchUser: AsyncHandler<string, User> = async (id) => {
  const response = await fetch(`/api/users/${id}`);
  return response.json();
};
```

## 🔍 에러 처리 모범 사례

### 1. Result 타입 패턴
```typescript
// ✅ 좋은 예: Result 타입으로 에러 처리
type Result<T, E = Error> = 
  | { success: true; data: T }
  | { success: false; error: E };

// 사용 예시
async function fetchUser(id: string): Promise<Result<User, string>> {
  try {
    const response = await fetch(`/api/users/${id}`);
    if (!response.ok) {
      return { success: false, error: `HTTP ${response.status}` };
    }
    const user = await response.json();
    return { success: true, data: user };
  } catch (error) {
    return { success: false, error: 'Network error' };
  }
}
```

### 2. 커스텀 에러 클래스
```typescript
// ✅ 좋은 예: 커스텀 에러 클래스
class AppError extends Error {
  constructor(
    message: string,
    public readonly statusCode: number,
    public readonly isOperational: boolean = true
  ) {
    super(message);
    this.name = 'AppError';
    Error.captureStackTrace(this, this.constructor);
  }
}

class ValidationError extends AppError {
  constructor(message: string) {
    super(message, 400);
    this.name = 'ValidationError';
  }
}

// 사용 예시
function validateUser(user: any): User {
  if (!user.username) {
    throw new ValidationError('Username is required');
  }
  return user;
}
```

## 🧪 테스트 모범 사례

### 1. 타입 안전한 테스트
```typescript
// ✅ 좋은 예: 타입 안전한 테스트
describe('UserService', () => {
  let userService: UserService;
  let mockUser: User;

  beforeEach(() => {
    userService = new UserService();
    mockUser = {
      id: '1',
      username: 'testuser',
      email: 'test@example.com',
      createdAt: new Date(),
      updatedAt: new Date(),
      isActive: true
    };
  });

  it('should create user with valid data', () => {
    const result = userService.createUser({
      username: 'newuser',
      email: 'new@example.com'
    });

    expect(result).toMatchObject({
      username: 'newuser',
      email: 'new@example.com',
      isActive: true
    });
    expect(result.id).toBeDefined();
    expect(result.createdAt).toBeInstanceOf(Date);
  });
});
```

### 2. Mock 타입 정의
```typescript
// ✅ 좋은 예: Mock 타입 정의
type MockUserService = {
  [K in keyof UserService]: jest.MockedFunction<UserService[K]>;
};

const createMockUserService = (): MockUserService => {
  return {
    createUser: jest.fn(),
    getUser: jest.fn(),
    updateUser: jest.fn(),
    deleteUser: jest.fn()
  };
};
```

## 🚀 성능 최적화 모범 사례

### 1. 타입 단순화
```typescript
// ✅ 좋은 예: 단순한 타입 사용
type UserId = string;
type Username = string;

// ❌ 나쁜 예: 복잡한 타입
type UserId = `user_${string}_${number}`;
```

### 2. 조건부 타입 활용
```typescript
// ✅ 좋은 예: 조건부 타입으로 유연한 API
type ApiResponse<T, E = never> = E extends never 
  ? { success: true; data: T }
  : { success: false; error: E };

// 사용 예시
type UserResponse = ApiResponse<User>;
type UserErrorResponse = ApiResponse<never, string>;
```

### 3. 유틸리티 타입 활용
```typescript
// ✅ 좋은 예: 유틸리티 타입 활용
interface CreateUserRequest {
  username: string;
  email: string;
  firstName?: string;
  lastName?: string;
}

// Partial: 모든 속성을 옵셔널로
type PartialUserRequest = Partial<CreateUserRequest>;

// Pick: 특정 속성만 선택
type UserCredentials = Pick<CreateUserRequest, 'username' | 'email'>;

// Omit: 특정 속성 제외
type UserProfile = Omit<CreateUserRequest, 'username'>;
```

## 📚 문서화 모범 사례

### 1. JSDoc 주석
```typescript
/**
 * 사용자를 생성합니다.
 * @param data - 사용자 생성에 필요한 데이터
 * @returns 생성된 사용자 정보
 * @throws {ValidationError} 유효하지 않은 데이터가 제공된 경우
 * @example
 * ```typescript
 * const user = await userService.createUser({
 *   username: 'john',
 *   email: 'john@example.com'
 * });
 * ```
 */
async createUser(data: CreateUserRequest): Promise<User> {
  // 구현
}
```

### 2. 타입 문서화
```typescript
/**
 * 사용자 상태를 나타내는 열거형
 * @enum {string}
 */
enum UserStatus {
  /** 활성 사용자 */
  ACTIVE = 'active',
  /** 비활성 사용자 */
  INACTIVE = 'inactive',
  /** 승인 대기 중인 사용자 */
  PENDING = 'pending'
}
```

## 🔧 도구 설정 모범 사례

### 1. ESLint 설정
```json
{
  "extends": [
    "@typescript-eslint/recommended",
    "@typescript-eslint/recommended-requiring-type-checking"
  ],
  "rules": {
    "@typescript-eslint/no-unused-vars": "error",
    "@typescript-eslint/no-explicit-any": "warn",
    "@typescript-eslint/explicit-function-return-type": "warn",
    "@typescript-eslint/no-non-null-assertion": "error",
    "@typescript-eslint/prefer-nullish-coalescing": "error",
    "@typescript-eslint/prefer-optional-chain": "error"
  }
}
```

### 2. Prettier 설정
```json
{
  "semi": true,
  "trailingComma": "es5",
  "singleQuote": true,
  "printWidth": 80,
  "tabWidth": 2,
  "useTabs": false
}
```

## 🎯 마이그레이션 체크리스트

### 기본 설정
- [ ] TypeScript 컴파일러 설정 완료
- [ ] ESLint 및 Prettier 설정 완료
- [ ] Jest 테스트 설정 완료
- [ ] Docker 빌드 설정 완료

### 타입 정의
- [ ] 기본 타입 정의 완료
- [ ] 인터페이스 정의 완료
- [ ] 열거형 정의 완료
- [ ] 유틸리티 타입 활용

### 코드 품질
- [ ] 타입 가드 구현
- [ ] 에러 처리 개선
- [ ] 테스트 코드 작성
- [ ] 문서화 완료

### 성능 최적화
- [ ] 타입 단순화
- [ ] 불필요한 타입 단언 제거
- [ ] 컴파일 시간 최적화
- [ ] 번들 크기 최적화

---

**TypeScript 모범 사례 가이드 완성일**: 2024년 9월 23일  
**적용 대상**: GitHub Actions Demo 프로젝트
