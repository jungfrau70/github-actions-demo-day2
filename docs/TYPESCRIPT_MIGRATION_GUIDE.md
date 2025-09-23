# 🔄 TypeScript 마이그레이션 가이드

## 📋 개요

이 문서는 현재 JavaScript로 작성된 GitHub Actions Demo 프로젝트를 TypeScript로 마이그레이션하는 방법을 설명합니다.

## 🎯 마이그레이션 목표

- **타입 안전성**: 컴파일 타임에 오류 검출
- **개발자 경험**: 향상된 IDE 지원 및 자동완성
- **코드 품질**: 더 나은 코드 구조 및 유지보수성
- **팀 협업**: 명확한 인터페이스 및 API 정의

## 📁 TypeScript 프로젝트 구조

```
github-actions-demo/
├── src/
│   ├── types/                   # TypeScript 타입 정의
│   │   ├── index.ts
│   │   ├── api.ts
│   │   └── database.ts
│   ├── controllers/             # 컨트롤러
│   │   ├── healthController.ts
│   │   └── apiController.ts
│   ├── services/                # 비즈니스 로직
│   │   ├── healthService.ts
│   │   └── metricsService.ts
│   ├── middleware/              # 미들웨어
│   │   ├── metrics.ts
│   │   └── errorHandler.ts
│   ├── routes/                  # 라우트 정의
│   │   ├── index.ts
│   │   ├── health.ts
│   │   └── api.ts
│   ├── config/                  # 설정
│   │   ├── database.ts
│   │   └── environment.ts
│   ├── utils/                   # 유틸리티 함수
│   │   ├── logger.ts
│   │   └── validation.ts
│   └── app.ts                   # 메인 애플리케이션
├── tests/
│   ├── unit/                    # 단위 테스트
│   │   ├── controllers/
│   │   ├── services/
│   │   └── utils/
│   └── integration/             # 통합 테스트
│       ├── api.test.ts
│       └── health.test.ts
├── dist/                        # 컴파일된 JavaScript 파일
├── tsconfig.json                # TypeScript 설정
├── tsconfig.test.json           # 테스트용 TypeScript 설정
├── package.json                 # 의존성 및 스크립트
└── Dockerfile.ts                # TypeScript용 Dockerfile
```

## 🔧 TypeScript 설정

### 1. package.json 업데이트
```json
{
  "name": "github-actions-demo",
  "version": "1.0.0",
  "description": "GitHub Actions CI/CD Demo with TypeScript",
  "main": "dist/app.js",
  "scripts": {
    "build": "tsc",
    "build:watch": "tsc --watch",
    "start": "node dist/app.js",
    "dev": "ts-node-dev --respawn --transpile-only src/app.ts",
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage",
    "lint": "eslint src/**/*.ts",
    "lint:fix": "eslint src/**/*.ts --fix",
    "type-check": "tsc --noEmit"
  },
  "dependencies": {
    "express": "^4.18.2",
    "prom-client": "^15.0.0",
    "cors": "^2.8.5",
    "helmet": "^7.1.0",
    "compression": "^1.7.4"
  },
  "devDependencies": {
    "@types/express": "^4.17.21",
    "@types/node": "^20.10.0",
    "@types/cors": "^2.8.17",
    "@types/compression": "^1.7.5",
    "@types/jest": "^29.5.8",
    "@types/supertest": "^2.0.16",
    "typescript": "^5.3.0",
    "ts-node": "^10.9.0",
    "ts-node-dev": "^2.0.0",
    "jest": "^29.7.0",
    "ts-jest": "^29.1.1",
    "supertest": "^6.3.3",
    "@typescript-eslint/eslint-plugin": "^6.13.0",
    "@typescript-eslint/parser": "^6.13.0",
    "eslint": "^8.54.0"
  }
}
```

### 2. tsconfig.json 설정
```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "lib": ["ES2020"],
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "removeComments": true,
    "noImplicitAny": true,
    "noImplicitReturns": true,
    "noImplicitThis": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "exactOptionalPropertyTypes": true,
    "noImplicitOverride": true,
    "noPropertyAccessFromIndexSignature": true,
    "noUncheckedIndexedAccess": true,
    "baseUrl": "./src",
    "paths": {
      "@/*": ["*"],
      "@/types/*": ["types/*"],
      "@/controllers/*": ["controllers/*"],
      "@/services/*": ["services/*"],
      "@/middleware/*": ["middleware/*"],
      "@/routes/*": ["routes/*"],
      "@/config/*": ["config/*"],
      "@/utils/*": ["utils/*"]
    }
  },
  "include": [
    "src/**/*"
  ],
  "exclude": [
    "node_modules",
    "dist",
    "tests"
  ]
}
```

### 3. tsconfig.test.json 설정
```json
{
  "extends": "./tsconfig.json",
  "compilerOptions": {
    "outDir": "./dist-test",
    "rootDir": "./tests",
    "types": ["jest", "node"]
  },
  "include": [
    "tests/**/*",
    "src/**/*"
  ]
}
```

## 📝 타입 정의

### 1. 기본 타입 정의 (src/types/index.ts)
```typescript
// 환경 변수 타입
export interface Environment {
  NODE_ENV: 'development' | 'staging' | 'production';
  PORT: number;
  HOST: string;
  LOG_LEVEL: 'error' | 'warn' | 'info' | 'debug';
}

// API 응답 타입
export interface ApiResponse<T = any> {
  success: boolean;
  data?: T;
  error?: string;
  timestamp: string;
}

// 헬스 체크 응답 타입
export interface HealthCheckResponse {
  status: 'OK' | 'ERROR';
  uptime: number;
  timestamp: string;
  memory: {
    rss: number;
    heapTotal: number;
    heapUsed: number;
    external: number;
    arrayBuffers: number;
  };
  environment: string;
  version: string;
}

// 메트릭 타입
export interface MetricsData {
  http_requests_total: number;
  http_request_duration_seconds: number;
  memory_usage_bytes: number;
  cpu_usage_percent: number;
}

// 에러 타입
export interface AppError extends Error {
  statusCode: number;
  isOperational: boolean;
}
```

### 2. API 타입 정의 (src/types/api.ts)
```typescript
import { Request, Response } from 'express';

// 요청 타입
export interface TypedRequest<T = any> extends Request {
  body: T;
  params: Record<string, string>;
  query: Record<string, string>;
}

// 응답 타입
export interface TypedResponse<T = any> extends Response {
  json: (body: T) => this;
}

// API 상태 응답 타입
export interface ApiStatusResponse {
  message: string;
  service: string;
  status: 'running' | 'stopped' | 'error';
  version: string;
  uptime: number;
  environment: string;
}

// 메트릭 응답 타입
export interface MetricsResponse {
  metrics: string;
  timestamp: string;
}
```

### 3. 데이터베이스 타입 정의 (src/types/database.ts)
```typescript
// 데이터베이스 연결 설정
export interface DatabaseConfig {
  host: string;
  port: number;
  database: string;
  username: string;
  password: string;
  ssl?: boolean;
  pool?: {
    min: number;
    max: number;
    idle: number;
  };
}

// 사용자 타입 (향후 확장용)
export interface User {
  id: string;
  username: string;
  email: string;
  createdAt: Date;
  updatedAt: Date;
}

// 로그 타입
export interface LogEntry {
  id: string;
  level: 'error' | 'warn' | 'info' | 'debug';
  message: string;
  timestamp: Date;
  metadata?: Record<string, any>;
}
```

## 🏗️ 애플리케이션 구조

### 1. 메인 애플리케이션 (src/app.ts)
```typescript
import express, { Application, Request, Response, NextFunction } from 'express';
import cors from 'cors';
import helmet from 'helmet';
import compression from 'compression';
import { config } from '@/config/environment';
import { errorHandler } from '@/middleware/errorHandler';
import { metricsMiddleware } from '@/middleware/metrics';
import { logger } from '@/utils/logger';
import { healthRoutes } from '@/routes/health';
import { apiRoutes } from '@/routes/api';

class App {
  public app: Application;

  constructor() {
    this.app = express();
    this.initializeMiddlewares();
    this.initializeRoutes();
    this.initializeErrorHandling();
  }

  private initializeMiddlewares(): void {
    this.app.use(helmet());
    this.app.use(cors());
    this.app.use(compression());
    this.app.use(express.json());
    this.app.use(express.urlencoded({ extended: true }));
    this.app.use(metricsMiddleware);
  }

  private initializeRoutes(): void {
    this.app.get('/', this.homeHandler);
    this.app.use('/health', healthRoutes);
    this.app.use('/api', apiRoutes);
    this.app.use('/metrics', this.metricsHandler);
  }

  private initializeErrorHandling(): void {
    this.app.use(errorHandler);
  }

  private homeHandler = (req: Request, res: Response): void => {
    res.send(`
      <!DOCTYPE html>
      <html>
      <head>
          <title>GitHub Actions Demo - TypeScript</title>
          <style>
              body { font-family: Arial, sans-serif; margin: 40px; }
              .container { max-width: 800px; margin: 0 auto; }
              .header { color: #333; border-bottom: 2px solid #007acc; padding-bottom: 10px; }
              .info { background: #f5f5f5; padding: 20px; border-radius: 5px; margin: 20px 0; }
              .endpoint { background: #e8f4fd; padding: 10px; margin: 10px 0; border-left: 4px solid #007acc; }
          </style>
      </head>
      <body>
          <div class="container">
              <h1 class="header">🚀 GitHub Actions Demo - TypeScript</h1>
              <div class="info">
                  <h2>CI/CD 실습 애플리케이션</h2>
                  <p><strong>버전:</strong> 2.0.0</p>
                  <p><strong>환경:</strong> ${config.NODE_ENV}</p>
                  <p><strong>시간:</strong> ${new Date().toISOString()}</p>
                  <p><strong>언어:</strong> TypeScript</p>
              </div>
              <div class="endpoint">
                  <h3>📊 사용 가능한 엔드포인트:</h3>
                  <ul>
                      <li><a href="/health">/health</a> - 헬스 체크</li>
                      <li><a href="/api/status">/api/status</a> - API 상태</li>
                      <li><a href="/metrics">/metrics</a> - Prometheus 메트릭</li>
                  </ul>
              </div>
          </div>
      </body>
      </html>
    `);
  };

  private metricsHandler = (req: Request, res: Response): void => {
    // 메트릭 처리 로직
    res.set('Content-Type', 'text/plain');
    res.send('# Metrics placeholder');
  };

  public listen(): void {
    this.app.listen(config.PORT, config.HOST, () => {
      logger.info(`🚀 Server running on http://${config.HOST}:${config.PORT}`);
      logger.info(`📊 Environment: ${config.NODE_ENV}`);
      logger.info(`🔧 TypeScript: Enabled`);
    });
  }
}

// 서버 시작
if (require.main === module) {
  const app = new App();
  app.listen();
}

export default App;
```

### 2. 헬스 체크 컨트롤러 (src/controllers/healthController.ts)
```typescript
import { Request, Response } from 'express';
import { HealthCheckResponse } from '@/types';
import { healthService } from '@/services/healthService';
import { logger } from '@/utils/logger';

export class HealthController {
  public async getHealth(req: Request, res: Response): Promise<void> {
    try {
      const healthData: HealthCheckResponse = await healthService.getHealthData();
      res.status(200).json(healthData);
    } catch (error) {
      logger.error('Health check failed:', error);
      res.status(500).json({
        status: 'ERROR',
        uptime: process.uptime(),
        timestamp: new Date().toISOString(),
        memory: process.memoryUsage(),
        environment: process.env.NODE_ENV || 'development',
        version: '2.0.0'
      });
    }
  }
}

export const healthController = new HealthController();
```

### 3. 헬스 체크 서비스 (src/services/healthService.ts)
```typescript
import { HealthCheckResponse } from '@/types';
import { config } from '@/config/environment';

export class HealthService {
  public async getHealthData(): Promise<HealthCheckResponse> {
    const memoryUsage = process.memoryUsage();
    
    return {
      status: 'OK',
      uptime: process.uptime(),
      timestamp: new Date().toISOString(),
      memory: {
        rss: memoryUsage.rss,
        heapTotal: memoryUsage.heapTotal,
        heapUsed: memoryUsage.heapUsed,
        external: memoryUsage.external,
        arrayBuffers: memoryUsage.arrayBuffers
      },
      environment: config.NODE_ENV,
      version: '2.0.0'
    };
  }
}

export const healthService = new HealthService();
```

### 4. 환경 설정 (src/config/environment.ts)
```typescript
import { Environment } from '@/types';

export const config: Environment = {
  NODE_ENV: (process.env.NODE_ENV as Environment['NODE_ENV']) || 'development',
  PORT: parseInt(process.env.PORT || '3000', 10),
  HOST: process.env.HOST || '0.0.0.0',
  LOG_LEVEL: (process.env.LOG_LEVEL as Environment['LOG_LEVEL']) || 'info'
};

// 환경 변수 검증
export const validateEnvironment = (): void => {
  const requiredEnvVars = ['NODE_ENV', 'PORT'];
  
  for (const envVar of requiredEnvVars) {
    if (!process.env[envVar]) {
      throw new Error(`Missing required environment variable: ${envVar}`);
    }
  }
};
```

### 5. 로거 유틸리티 (src/utils/logger.ts)
```typescript
import { config } from '@/config/environment';

export interface Logger {
  info(message: string, meta?: any): void;
  warn(message: string, meta?: any): void;
  error(message: string, meta?: any): void;
  debug(message: string, meta?: any): void;
}

class AppLogger implements Logger {
  private logLevel: string;

  constructor() {
    this.logLevel = config.LOG_LEVEL;
  }

  private shouldLog(level: string): boolean {
    const levels = ['error', 'warn', 'info', 'debug'];
    return levels.indexOf(level) <= levels.indexOf(this.logLevel);
  }

  private formatMessage(level: string, message: string, meta?: any): string {
    const timestamp = new Date().toISOString();
    const metaStr = meta ? ` ${JSON.stringify(meta)}` : '';
    return `[${timestamp}] ${level.toUpperCase()}: ${message}${metaStr}`;
  }

  public info(message: string, meta?: any): void {
    if (this.shouldLog('info')) {
      console.log(this.formatMessage('info', message, meta));
    }
  }

  public warn(message: string, meta?: any): void {
    if (this.shouldLog('warn')) {
      console.warn(this.formatMessage('warn', message, meta));
    }
  }

  public error(message: string, meta?: any): void {
    if (this.shouldLog('error')) {
      console.error(this.formatMessage('error', message, meta));
    }
  }

  public debug(message: string, meta?: any): void {
    if (this.shouldLog('debug')) {
      console.debug(this.formatMessage('debug', message, meta));
    }
  }
}

export const logger = new AppLogger();
```

## 🧪 테스트 설정

### 1. Jest 설정 (jest.config.js)
```javascript
module.exports = {
  preset: 'ts-jest',
  testEnvironment: 'node',
  roots: ['<rootDir>/src', '<rootDir>/tests'],
  testMatch: ['**/__tests__/**/*.ts', '**/?(*.)+(spec|test).ts'],
  transform: {
    '^.+\\.ts$': 'ts-jest',
  },
  collectCoverageFrom: [
    'src/**/*.ts',
    '!src/**/*.d.ts',
    '!src/**/*.test.ts',
    '!src/**/*.spec.ts',
  ],
  coverageDirectory: 'coverage',
  coverageReporters: ['text', 'lcov', 'html'],
  moduleNameMapping: {
    '^@/(.*)$': '<rootDir>/src/$1',
  },
  setupFilesAfterEnv: ['<rootDir>/tests/setup.ts'],
};
```

### 2. 테스트 예제 (tests/unit/controllers/healthController.test.ts)
```typescript
import request from 'supertest';
import express from 'express';
import { healthController } from '@/controllers/healthController';
import { healthService } from '@/services/healthService';

// Mock the health service
jest.mock('@/services/healthService');
const mockedHealthService = healthService as jest.Mocked<typeof healthService>;

describe('HealthController', () => {
  let app: express.Application;

  beforeEach(() => {
    app = express();
    app.use(express.json());
    app.get('/health', healthController.getHealth.bind(healthController));
  });

  afterEach(() => {
    jest.clearAllMocks();
  });

  describe('GET /health', () => {
    it('should return health data successfully', async () => {
      const mockHealthData = {
        status: 'OK' as const,
        uptime: 123.45,
        timestamp: '2024-01-01T00:00:00.000Z',
        memory: {
          rss: 1000000,
          heapTotal: 500000,
          heapUsed: 300000,
          external: 10000,
          arrayBuffers: 5000
        },
        environment: 'test',
        version: '2.0.0'
      };

      mockedHealthService.getHealthData.mockResolvedValue(mockHealthData);

      const response = await request(app)
        .get('/health')
        .expect(200);

      expect(response.body).toEqual(mockHealthData);
      expect(mockedHealthService.getHealthData).toHaveBeenCalledTimes(1);
    });

    it('should handle service errors gracefully', async () => {
      mockedHealthService.getHealthData.mockRejectedValue(new Error('Service error'));

      const response = await request(app)
        .get('/health')
        .expect(500);

      expect(response.body.status).toBe('ERROR');
      expect(response.body.environment).toBeDefined();
    });
  });
});
```

## 🐳 Docker 설정

### 1. TypeScript용 Dockerfile (Dockerfile.ts)
```dockerfile
# 멀티스테이지 빌드
FROM node:18-alpine AS builder

WORKDIR /app

# 의존성 설치
COPY package*.json ./
RUN npm ci --only=production

# TypeScript 설치
RUN npm install -g typescript

# 소스 코드 복사
COPY . .

# TypeScript 컴파일
RUN npm run build

# 프로덕션 스테이지
FROM node:18-alpine AS runtime

WORKDIR /app

# 프로덕션 의존성만 설치
COPY package*.json ./
RUN npm ci --only=production && npm cache clean --force

# 컴파일된 JavaScript 파일 복사
COPY --from=builder /app/dist ./dist

# 사용자 생성
RUN addgroup -g 1001 -S nodejs
RUN adduser -S nodejs -u 1001

# 권한 설정
RUN chown -R nodejs:nodejs /app
USER nodejs

# 포트 노출
EXPOSE 3000

# 헬스 체크
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD node -e "require('http').get('http://localhost:3000/health', (res) => { process.exit(res.statusCode === 200 ? 0 : 1) })"

# 애플리케이션 시작
CMD ["node", "dist/app.js"]
```

### 2. Docker Compose 업데이트
```yaml
# docker-compose.ts.yml
version: '3.8'

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile.ts
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - PORT=3000
      - HOST=0.0.0.0
      - LOG_LEVEL=info
    depends_on:
      - postgres
      - redis
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s

  postgres:
    image: postgres:15-alpine
    environment:
      - POSTGRES_DB=github_actions_demo
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
    volumes:
      - postgres_data:/var/lib/postgresql/data
    restart: unless-stopped

  redis:
    image: redis:7-alpine
    restart: unless-stopped

volumes:
  postgres_data:
```

## 🔄 마이그레이션 단계

### 1단계: 기본 설정
```bash
# TypeScript 의존성 설치
npm install -D typescript @types/node @types/express

# TypeScript 설정 파일 생성
npx tsc --init

# ESLint 설정
npm install -D @typescript-eslint/eslint-plugin @typescript-eslint/parser
```

### 2단계: 점진적 마이그레이션
```bash
# 1. 타입 정의부터 시작
mkdir -p src/types
# types/index.ts 파일 생성

# 2. 기존 JavaScript 파일을 .ts로 변경
mv src/app.js src/app.ts

# 3. 타입 에러 수정
npm run type-check

# 4. 테스트 추가
npm install -D @types/jest ts-jest
```

### 3단계: 완전 마이그레이션
```bash
# 모든 파일을 TypeScript로 변환
find src -name "*.js" -exec sh -c 'mv "$1" "${1%.js}.ts"' _ {} \;

# 빌드 및 테스트
npm run build
npm run test
```

## 🚀 GitHub Actions 업데이트

### TypeScript 빌드 워크플로우
```yaml
# .github/workflows/typescript-ci.yml
name: TypeScript CI/CD

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  type-check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Type check
        run: npm run type-check
      
      - name: Lint
        run: npm run lint

  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run tests
        run: npm run test:coverage
      
      - name: Upload coverage
        uses: codecov/codecov-action@v3

  build:
    runs-on: ubuntu-latest
    needs: [type-check, test]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Build TypeScript
        run: npm run build
      
      - name: Build Docker image
        run: docker build -f Dockerfile.ts -t ${{ secrets.DOCKER_USERNAME }}/github-actions-demo:typescript .
```

## 📊 마이그레이션 효과

### 개발자 경험 개선
- **자동완성**: IDE에서 더 정확한 자동완성 제공
- **리팩토링**: 안전한 코드 리팩토링 지원
- **오류 검출**: 컴파일 타임에 오류 발견

### 코드 품질 향상
- **타입 안전성**: 런타임 오류 감소
- **문서화**: 타입이 곧 문서 역할
- **유지보수성**: 더 나은 코드 구조

### 팀 협업 강화
- **명확한 인터페이스**: API 계약 명확화
- **코드 리뷰**: 더 효과적인 코드 리뷰
- **온보딩**: 새로운 팀원의 빠른 적응

## 🎯 다음 단계

1. **점진적 마이그레이션**: 기존 코드를 단계적으로 TypeScript로 변환
2. **고급 타입 활용**: 제네릭, 유니온 타입 등 고급 기능 활용
3. **성능 최적화**: TypeScript 컴파일 최적화
4. **도구 통합**: 더 나은 개발 도구 통합

---

**TypeScript 마이그레이션 가이드 완성일**: 2024년 9월 23일  
**적용 대상**: GitHub Actions Demo 프로젝트
