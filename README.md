# Cs9labs Modern TypeScript Style Guide
### A developer-friendly and AI-aligned style guide for modern frontend development.

This guide provides clean, consistent, and easy-to-understand standards for writing TypeScript in modern frontend projects (React, Next.js, Vite). 

#### Why a new style guide?
Unlike standard style guides, this guide is designed with **two audiences** in mind:
1. **Human Developers:** Easy to read, clear rules, and no overly complex tricks.
2. **AI Coding Agents (LLMs):** Predictable patterns, explicit type declarations, and simple structures. When AI agents write code following this guide, they make fewer reasoning mistakes, suffer less hallucination, and produce clean code on the first try.

---

## Table of Contents
1. [Core Philosophy](#1-core-philosophy)
2. [Naming Conventions](#2-naming-conventions)
3. [Types & Interfaces](#3-types--interfaces)
4. [Modern Frontend & React Patterns](#4-modern-frontend--react-patterns)
5. [Control Flow & Language Features](#5-control-flow--language-features)
6. [AI-Friendly Best Practices](#6-ai-friendly-best-practices)

---

## 1. Core Philosophy

* **Predictability over Cleverness:** Code should be easy to read and understand. Avoid complex TypeScript features (like deep nested generic types) if a simple structure can do the job.
* **Explicit is Better than Implicit:** Write explicit type definitions for entry points and public functions. Do not rely 100% on automatic type inference.
* **Single Responsibility:** Keep files small. One file should do one thing (e.g., one component, one hook, or one utility function).

---

## 2. Naming Conventions

### 2.1 File & Directory Names
* **React Components:** Use `PascalCase`.
  ```
  // Good
  UserCard.tsx
  SidebarToggle.tsx

  // Bad
  user-card.tsx
  sidebarToggle.tsx
  ```
* **Hooks, Utilities, and Helpers:** Use `camelCase`.
  ```
  // Good
  useAuth.ts
  formatCurrency.ts

  // Bad
  UseAuth.ts
  FormatCurrency.ts
  format-currency.ts
  ```

### 2.2 Variables & Functions
* Use `camelCase` for variable and function names.
  ```typescript
  // Good
  const activeUserCount = 10;
  function calculateTotal(price: number, tax: number): number { ... }

  // Bad
  const active_user_count = 10;
  function CalculateTotal(price: number, tax: number) { ... }
  ```

### 2.3 Interfaces, Types & Generics
* Use `PascalCase` for Interfaces and Type Aliases.
* **Do not** prefix interfaces with `I`.
  ```typescript
  // Good
  interface UserProfile {
    id: string;
    name: string;
  }

  // Bad
  interface IUserProfile {
    id: string;
    name: string;
  }
  ```
* Use descriptive names for Generics. Avoid single-letter names like `T` or `U` unless it is a very simple, pure utility function.
  ```typescript
  // Good
  interface ApiResponse<TData> {
    data: TData;
    status: number;
  }

  // Bad
  interface ApiResponse<T> {
    data: T;
    status: number;
  }
  ```

---

## 3. Types & Interfaces

### 3.1 `interface` vs. `type`
* Use **`interface`** for objects and public API structures (because interfaces support extension and compile faster).
* Use **`type`** for unions, intersections, utility mappings, and primitive aliases.

```typescript
// Good: Use interface for object structures
interface User {
  id: string;
  email: string;
}

// Good: Use type for unions or combinations
type Role = 'admin' | 'editor' | 'viewer';
type UserWithRole = User & { role: Role };

// Bad: Using type for simple objects
type UserDetail = {
  id: string;
  email: string;
};
```

### 3.2 Explicit Return Types
* Always write explicit return types for exported functions, React components, and custom hooks.
* *Why?* This helps the compiler detect errors early and allows AI agents to understand the boundary interfaces of your code immediately.

```typescript
// Good
export function formatName(firstName: string, lastName: string): string {
  return `${firstName} ${lastName}`;
}

export function useCounter(): { count: number; increment: () => void } {
  const [count, setCount] = useState(0);
  const increment = () => setCount((c) => c + 1);
  return { count, increment };
}

// Bad
export function formatName(firstName: string, lastName: string) {
  return `${firstName} ${lastName}`; // Return type is implicit
}
```

### 3.3 Avoid `any`
* Do not use `any`. If you do not know the type of a value beforehand, use `unknown`.
* Using `unknown` forces you to check the type before using it, preventing runtime crashes.

```typescript
// Good
function processData(value: unknown): void {
  if (typeof value === 'string') {
    console.log(value.toUpperCase()); // Safe
  }
}

// Bad
function processData(value: any): void {
  console.log(value.toUpperCase()); // Will crash if value is not a string
}
```

### 3.4 Avoid Type Assertions (`as`)
* Avoid using `as` to override the compiler. Instead, use type guards, check for null/undefined, or use validation libraries like `Zod`.

```typescript
// Good: Safe check or validation
interface ApiUser {
  name: string;
}

function parseUser(data: unknown): ApiUser {
  if (data && typeof data === 'object' && 'name' in data) {
    return data as ApiUser; // Acceptable narrow assertion after check
  }
  throw new Error('Invalid user data');
}

// Bad: Blind assertion
const user = rawData as ApiUser; // Dangerous if rawData is null or different
```

---

## 4. Modern Frontend & React Patterns

### 4.1 Declaring Components
* Use standard function declarations instead of `React.FC` or `React.FunctionComponent`.
* *Why?* Plain functions are easier to read, work better with default parameters, and don't inject implicit children types.

```typescript
// Good
interface ButtonProps {
  label: string;
  onClick: () => void;
}

export function Button({ label, onClick }: ButtonProps): React.JSX.Element {
  return <button onClick={onClick}>{label}</button>;
}

// Bad
export const Button: React.FC<ButtonProps> = ({ label, onClick }) => {
  return <button onClick={onClick}>{label}</button>;
};
```

### 4.2 Props Destructuring
* Destructure React props directly in the component signature.
* Provide default values in the destructuring assignment.

```typescript
// Good
interface UserAvatarProps {
  imageUrl: string;
  size?: 'small' | 'large';
}

export function UserAvatar({ imageUrl, size = 'small' }: UserAvatarProps): React.JSX.Element {
  return <img src={imageUrl} className={`avatar-${size}`} alt="Avatar" />;
}

// Bad
export function UserAvatar(props: UserAvatarProps): React.JSX.Element {
  const size = props.size || 'small'; // Ad-hoc defaulting
  return <img src={props.imageUrl} className={`avatar-${size}`} alt="Avatar" />;
}
```

### 4.3 Hooks: Single Responsibility & Dependency Arrays
* Custom hooks should do one specific job.
* Always fill out the full dependency array for `useEffect`, `useCallback`, and `useMemo`. Never leave them empty or ignore linter warnings (`eslint-disable-next-line react-hooks/exhaustive-deps`).

```typescript
// Good
const fetchUser = useCallback(() => {
  if (userId) {
    loadUser(userId);
  }
}, [userId, loadUser]); // All external values are listed in dependencies
```

---

## 5. Control Flow & Language Features

### 5.1 Optional Chaining & Nullish Coalescing
* Use optional chaining (`?.`) and nullish coalescing (`??`) to handle safe fallback values.
* Avoid the logical OR operator (`||`) for default values because it treats `0` or `""` (empty string) as falsy.

```typescript
// Good
const displayName = user?.profile?.nickname ?? 'Anonymous';
const score = game?.score ?? 0; // Safe: if score is 0, it stays 0

// Bad
const displayName = user && user.profile ? user.profile.nickname : 'Anonymous';
const score = game?.score || 0; // If score is 0, it falls back to 0 (correct, but unsafe if score can be 0)
const text = inputMessage || 'Default message'; // Bad: if message is empty string "", it displays 'Default message'
```

### 5.2 Avoid `enum`
* Do not use TypeScript `enum`. Enums create extra code during compilation and do not match standard JavaScript behavior.
* Instead, use `as const` object mappings or simple Union Types.

```typescript
// Good: Union Types
type Status = 'pending' | 'success' | 'failed';

// Good: Read-only config object
const PROJECT_STATUS = {
  PENDING: 'pending',
  SUCCESS: 'success',
  FAILED: 'failed',
} as const;

type ProjectStatus = typeof PROJECT_STATUS[keyof typeof PROJECT_STATUS];

// Bad: TypeScript Enum
enum StatusEnum {
  Pending = 'pending',
  Success = 'success',
  Failed = 'failed',
}
```

### 5.3 Conditional Rendering in JSX
* Avoid using `&&` for conditional rendering when the left-hand side could be a number or string.
* *Why?* In React, `{0 && <Component />}` will print `0` to the screen. Use ternary operators or convert to explicit boolean.

```typescript
// Good
{hasItems ? <ItemList /> : null}
{Boolean(itemCount) && <Counter count={itemCount} />}

// Bad
{itemCount && <Counter count={itemCount} />} // Renders '0' on screen if itemCount is 0
```

---

## 6. AI-Friendly Best Practices

AI Coding Agents write better code when the codebase is modular, predictable, and strictly structured. Follow these guidelines to ensure AI writes clean code for you:

### 6.1 Small File Limits
* Keep files under **250 lines of code**.
* If a component or helper gets too long, split it into smaller files.
* *Why?* AI models have limited context windows and perform significantly better when analyzing a single, tightly-scoped file.

### 6.2 Strict TypeScript Config
* Always enable `strict: true` in your `tsconfig.json`.
* Preventing implicit `any`, forcing strict null checks, and enforcing clean interfaces will guide the AI to generate robust code without guessing.

### 6.3 Standard API Response Typings
* Define clear contracts for API payloads. AI models can perfectly generate integration code if they have access to exact JSON payloads typed as TypeScript interfaces.

```typescript
interface UserResponse {
  id: string;
  name: string;
  createdAt: string;
}

// AI can immediately and correctly generate fetchers and mock data based on this contract.
```
