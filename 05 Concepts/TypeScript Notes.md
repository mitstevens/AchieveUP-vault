---
tags: [concept, frontend]
---

# TypeScript Notes

TypeScript = JavaScript + static types. The compiler catches "this object has no field `riskLevel`" at build time instead of runtime.

## What you'll actually use here
```ts
// interfaces describe object shapes — the data model lives in src/types/index.ts
interface User { id: string; email: string; role: 'instructor' | 'admin'; }

// union literal types — used everywhere for enums
riskLevel: 'low' | 'medium' | 'high'

// generics — typed API responses in services/api.ts
login(data): Promise<AxiosResponse<{ token: string; user: User }>>

// typing a component (props in <>)
const Button: React.FC<ButtonProps> = ({ children }) => ...

// optional fields and escape hatches
earnedAt?: string        // may be absent
const [courses, setCourses] = useState<any[]>([]);  // `any` = types off ⚠️ common in this repo
```

## Project specifics
- **`src/types/index.ts` is the de-facto data model** for the whole system — Mongo has no schema, so these interfaces are the best documentation of what backend responses look like. Read it early.
- `.tsx` = TypeScript + JSX (components); `.ts` = plain TypeScript (api.ts, types).
- `tsconfig.json` configures the compiler; CRA runs type-checking during `npm start`/`build`.
- The codebase leans on `any` in places (e.g. `StudentProgress`) — fine to tighten as you touch files, but don't mass-refactor.

Related: [[React Concepts]] · [[Frontend API Layer]]
