# Evidencias (10-quality-testing)

- **Jest config – `jest.config.js`**

```js
10 setupFilesAfterEnv: ['<rootDir>/jest.setup.js'],
11 testEnvironment: 'jest-environment-jsdom',
30 collectCoverageFrom: ['components/**/*.{js,jsx,ts,tsx}', 'lib/**/*.{js,jsx,ts,tsx}', 'app/**/*.{js,jsx,ts,tsx}', 'store/**/*.{js,jsx,ts,tsx}'],
41 coverageThreshold: { global: { branches: 70, functions: 70, lines: 70, statements: 70 }},
50 testTimeout: 10000
```

- **Testing Library setup – `jest.setup.js`**

```js
2 import '@testing-library/jest-dom';
```

- **Scripts y deps – `package.json`**

```json
"scripts": { "test": "jest", "test:coverage": "jest --coverage", "type-check": "tsc --noEmit" }
"devDependencies": { "jest": "^30.0.5", "@testing-library/react": "^16.3.0" }
```

- **Tests UI – `__tests__/components/sections.test.tsx`**

```tsx
12 global.fetch = jest.fn()
35 it('renders loading state initially', ...)
67 (fetch as jest.Mock).mockResolvedValueOnce({ ok: true, json: ... })
```

- **Tests API – `__tests__/api/admin.orders.get.test.ts`**

```ts
9 jest.mock('@/lib/auth-middleware', ...)
16 jest.mock('@/lib/mongodb', ...)
162 jest.mock('@/models/Order', ... makeChain ...)
224 const res = (await GET(req as any)) as Response
```

- **Integración DB – `__tests__/integration/scheduler.integration.test.ts`**

```ts
1 import { MongoMemoryServer } from 'mongodb-memory-server'
17 mongod = await MongoMemoryServer.create()
19 process.env.MONGODB_URI = uri; await connectToDatabase()
9 jest.setTimeout(30000)
```

- **ESLint – `.eslintrc.json`**

```json
"extends": ["next/core-web-vitals"], "rules": { "no-unused-vars": ["error", {"argsIgnorePattern": "^_"}], "no-debugger": "warn" }
```

- **TS Strict – `tsconfig.json`**

```json
"strict": true, "noEmit": true, "moduleResolution": "bundler"
```
