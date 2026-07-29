---
name: typescript-patterns
description: TypeScript patterns and best practices including satisfies operator, branded types, discriminated unions, template literal types, type predicates, and modern TypeScript idioms inspired by Matt Pocock's Total TypeScript.
metadata:
  origin: ECC
---

# TypeScript Patterns

Modern TypeScript patterns inspired by Matt Pocock's Total TypeScript for building type-safe, expressive, and maintainable code.

## When to Activate

- Writing or reviewing TypeScript files (`.ts`, `.tsx`)
- Designing type systems, generics, or utility types
- Validating API boundaries and data contracts
- Replacing runtime validation with compile-time guarantees
- Choosing between type narrowing strategies
- Modeling complex domain logic with the type system
- Working with Zod for runtime + compile-time safety

## Core Principles

### 1. Think in Types, Not Just Values

TypeScript's type system is a programming language of its own. Use it to encode business rules, not just shape-check data.

### 2. Inference > Explicit Annotation

Let TypeScript infer types where possible. Add explicit annotations only at boundaries (function signatures, API responses, public exports).

### 3. Narrow as Early as Possible

Narrow unknown or wide types at the earliest point — ideally at the boundary where data enters your system.

### 4. Composition Over Primitives

Build complex types by composing smaller, reusable type utilities. Prefer mapped types and conditional types over hand-written permutations.

---

## 1. `satisfies` Operator (TypeScript 4.9+)

### What It Is

`satisfies` validates that a value's type matches a constraint *without* widening the inferred type. The variable keeps its narrow inferred type while being checked against the constraint.

### `satisfies` vs `as`

| Approach | Effect |
|---|---|
| `value as SomeType` | Asserts the type — erases the original narrow type. Dangerous if wrong. |
| `value satisfies SomeType` | Checks the type — preserves the original narrow type. Safe. |

```typescript
// PASS: satisfies preserves the literal type
const palette = {
  red: [255, 0, 0],
  green: "#00ff00",
  blue: [0, 0, 255],
} satisfies Record<string, string | number[]>;

// palette.red is number[] (not string | number[])
palette.red.map(x => x.toFixed(2));

// FAIL: 'as' widens and loses information
const palette2 = {
  red: [255, 0, 0],
  green: "#00ff00",
} as Record<string, string | number[]>;

// palette2.red is string | number[] — needs narrowing
```

### Arrays and Narrowed Types

```typescript
// PASS: satisfies validates element type without widening to string[]
const colors = ["red", "green", "blue"] satisfies readonly string[];
type Color = (typeof colors)[number]; // "red" | "green" | "blue"

// FAIL: 'as const' is narrower but doesn't validate shape
const colors2 = ["red", 42] as const; // no error — loses validation
```

### Object Patterns

```typescript
type Color = "red" | "green" | "blue";

// PASS: each value must be a valid RGB tuple
const hexValues = {
  red: "#ff0000",
  green: "#00ff00",
  blue: "#0000ff",
} satisfies Record<Color, string>;

// FAIL: wrong key would error
// const bad = { red: "#ff0000", purple: "#800080" } satisfies Record<Color, string>;
```

---

## 2. Branded Types (Nominal Typing)

### The Pattern

TypeScript uses structural typing. Branded types add a phantom property to create nominal-like distinctions.

```typescript
// PASS: branded type for entity IDs
type UserId = string & { readonly __brand: "UserId" };
type PostId = string & { readonly __brand: "PostId" };

function createUser(id: UserId) {
  // ...
}

// PASS: brands prevent mixing entity IDs
const userId = "abc-123" as UserId;
const postId = "abc-123" as PostId;

createUser(userId); // OK
// createUser(postId); // FAIL: Type 'PostId' is not assignable to type 'UserId'

// FAIL: plain string is not assignable
// createUser("some-string");
```

### When to Use

- Entity IDs: `UserId`, `OrderId`, `ProductId`
- Currency: `type EUR = number & { readonly __brand: "EUR" }`
- Validated emails: `type Email = string & { readonly __brand: "Email" }`
- Any case where two structurally identical types must be treated differently

### Helper Factory

```typescript
// PASS: factory with validation and branding
function createUserId(value: string): UserId {
  if (!/^[a-f0-9-]+$/i.test(value)) {
    throw new Error("Invalid user ID format");
  }
  return value as UserId;
}
```

### Zod Branded Types

```typescript
import { z } from "zod";

// PASS: Zod branded type
const UserIdSchema = z.string().uuid().brand("UserId");
type UserId = z.infer<typeof UserIdSchema>;

const PostIdSchema = z.string().uuid().brand("PostId");
type PostId = z.infer<typeof PostIdSchema>;

function getUser(id: UserId) {
  // ...
}

const raw = "550e8400-e29b-41d4-a716-446655440000";
const id = UserIdSchema.parse(raw);
getUser(id); // OK

// FAIL: different brand
// getUser(postId as PostId);
```

---

## 3. Discriminated Unions

### Result Pattern

```typescript
// PASS: Result type with discriminated union
type Result<T, E> =
  | { success: true; data: T }
  | { success: false; error: E };

function parseJson(input: string): Result<unknown, SyntaxError> {
  try {
    return { success: true, data: JSON.parse(input) };
  } catch (e) {
    return { success: false, error: e as SyntaxError };
  }
}
```

### Exhaustive Narrowing with Switch

```typescript
// PASS: switch narrows the discriminated union exhaustively
type Shape =
  | { kind: "circle"; radius: number }
  | { kind: "square"; side: number }
  | { kind: "triangle"; base: number; height: number };

function area(shape: Shape): number {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "square":
      return shape.side ** 2;
    case "triangle":
      return (shape.base * shape.height) / 2;
    default:
      return _exhaustive(shape);
  }
}

function _exhaustive(x: never): never {
  throw new Error(`Unexpected value: ${x}`);
}

// FAIL: missing a case means no compile error if _exhaustive is missing
// function areaIncomplete(shape: Shape): number {
//   switch (shape.kind) {
//     case "circle":
//       return Math.PI * shape.radius ** 2;
//     // forgot square and triangle — no error at compile time
//   }
// }
```

### Never Check for Exhaustion

```typescript
// PASS: _exhaustive utility with never
function assertNever(x: never): never {
  throw new Error(`Unexpected value: ${x}`);
}

// PASS: adding a new member to the union forces updates everywhere
// type Shape =
//   | { kind: "circle"; radius: number }
//   | { kind: "square"; side: number }
//   | { kind: "triangle"; base: number; height: number }
//   | { kind: "rectangle"; width: number; height: number }; // adding this breaks area()
```

---

## 4. Type Predicates

### Basic Type Predicate

```typescript
// PASS: type predicate narrows the type for callers
interface User {
  id: string;
  name: string;
  email: string;
}

interface Admin {
  id: string;
  role: "admin";
  permissions: string[];
}

function isAdmin(user: User | Admin): user is Admin {
  return "role" in user && user.role === "admin";
}

// PASS: after the check, TypeScript knows the narrowed type
const users: (User | Admin)[] = [];
const admins = users.filter(isAdmin); // Admin[]
```

### Complex Narrowing with Custom Guards

```typescript
// PASS: multi-step type predicate
interface SuccessResponse<T> {
  status: "success";
  data: T;
}
interface ErrorResponse {
  status: "error";
  message: string;
}
type ApiResponse<T> = SuccessResponse<T> | ErrorResponse;

function isSuccess<T>(response: ApiResponse<T>): response is SuccessResponse<T> {
  return response.status === "success";
}

// PASS: type-safe response handling
async function fetchUser(id: string) {
  const response = await fetch(`/api/users/${id}`);
  const json: ApiResponse<User> = await response.json();

  if (isSuccess(json)) {
    return json.data; // User
  }
  throw new Error(json.message);
}
```

### Assertion Functions

```typescript
// PASS: assertion functions narrow after throwing
function assertIsString(value: unknown): asserts value is string {
  if (typeof value !== "string") {
    throw new Error("Expected string");
  }
}

function process(input: unknown) {
  assertIsString(input);
  // input is now string
  console.log(input.toUpperCase());
}

// FAIL: without assertion function, the type stays unknown
// function processBad(input: unknown) {
//   if (typeof input === "string") { /* ok here */ }
//   // outside the if, input is unknown again
// }
```

---

## 5. Template Literal Types

### Event Handler Pattern

```typescript
// PASS: template literal for event handlers
type EventName = "click" | "focus" | "blur";
type HandlerName = `on${Capitalize<EventName>}`;
// "onClick" | "onFocus" | "onBlur"

// FAIL: type-safe event handler props
type EventHandlers = {
  [K in HandlerName]: (event: Event) => void;
};
// { onClick: (event: Event) => void; onFocus: (event: Event) => void; ... }
```

### String Transformation Utilities

```typescript
// PASS: built-in string manipulation types
type Greeting = "hello-world";

type Upper = Uppercase<Greeting>;        // "HELLO-WORLD"
type Lower = Lowercase<Upper>;           // "hello-world"
type Capital = Capitalize<Greeting>;     // "Hello-world"
type Uncapital = Uncapitalize<Capital>;  // "hello-world"

// PASS: combining infer with template literals
type ExtractId<T extends string> =
  T extends `${string}_${infer Id}` ? Id : never;

type Test = ExtractId<"user_abc123">; // "abc123"
```

### CSS Property Pattern

```typescript
// PASS: mapped type with template literals
type CSSProperty = "margin" | "padding" | "border";
type CSSDirection = "top" | "right" | "bottom" | "left";

type CSSProperties = {
  [K in `${CSSProperty}${Capitalize<CSSDirection>}`]: string;
};
// {
//   marginTop: string; marginRight: string;
//   paddingTop: string; paddingRight: string;
//   ...
// }

// FAIL: manually listing all combinations is error-prone
// type ManualCSS = {
//   marginTop: string; marginRight: string;
//   // ... 12 properties, easy to miss one
// };
```

---

## 6. `as const` Assertions

### Immutable Tuples

```typescript
// PASS: as const infers literal tuple, not string[]
const COLORS = ["red", "green", "blue"] as const;
type Color = (typeof COLORS)[number]; // "red" | "green" | "blue"

// FAIL: without as const, array is typed string[]
const COLORS_BAD = ["red", "green", "blue"];
type ColorBad = (typeof COLORS_BAD)[number]; // string — too wide
```

### Enum-like Objects

```typescript
// PASS: as const with objects for type-safe enum alternative
const HTTP_STATUS = {
  OK: 200,
  CREATED: 201,
  BAD_REQUEST: 400,
  UNAUTHORIZED: 401,
  NOT_FOUND: 404,
  INTERNAL_SERVER_ERROR: 500,
} as const;

type HttpStatusCode = (typeof HTTP_STATUS)[keyof typeof HTTP_STATUS];
// 200 | 201 | 400 | 401 | 404 | 500

// PASS: discriminated union from as const object
const ROUTES = {
  HOME: "/",
  ABOUT: "/about",
  CONTACT: "/contact",
} as const;

type Route = (typeof ROUTES)[keyof typeof ROUTES]; // "/" | "/about" | "/contact"

// FAIL: without as const, values are just string
const ROUTES_BAD = { HOME: "/", ABOUT: "/about" };
type RouteBad = (typeof ROUTES_BAD)[keyof typeof ROUTES_BAD]; // string — no narrowing
```

### Narrowing with `as const` in Generics

```typescript
// PASS: const type parameters preserve literal types
function tuple<T extends readonly string[]>(...items: T): T {
  return items;
}

const result = tuple("a", "b", "c"); // readonly ["a", "b", "c"]

// FAIL: without const type parameter, literals widen
function tupleBad<T extends string[]>(...items: T): T {
  return items;
}
const resultBad = tupleBad("a", "b", "c"); // string[] — lost literals
```

---

## 7. Conditional Types + `infer`

### Basic Conditional Types

```typescript
// PASS: conditional type for extracting types
type IsString<T> = T extends string ? true : false;
type A = IsString<"hello">; // true
type B = IsString<42>;      // false

// PASS: filtering with conditional types
type ExtractStrings<T> = T extends string ? T : never;
type StringsOnly = ExtractStrings<"a" | 1 | "b" | 2>; // "a" | "b"
```

### `infer` Keyword

```typescript
// PASS: infer extracts types from within other types
type UnwrapPromise<T> = T extends Promise<infer U> ? U : T;
type A = UnwrapPromise<Promise<string>>; // string
type B = UnwrapPromise<number>;          // number

// PASS: extracting element type from an array
type ElementType<T> = T extends (infer U)[] ? U : never;
type C = ElementType<string[]>;  // string
type D = ElementType<number[]>;  // number
```

### Manual Implementation of Built-in Utilities

```typescript
// PASS: manual ReturnType
type MyReturnType<T extends (...args: any) => any> =
  T extends (...args: any) => infer R ? R : never;

function greet() { return "hello"; }
type GreetReturn = MyReturnType<typeof greet>; // string

// PASS: manual Parameters
type MyParameters<T extends (...args: any) => any> =
  T extends (...args: infer P) => any ? P : never;

function add(a: number, b: number) { return a + b; }
type AddParams = MyParameters<typeof add>; // [number, number]

// PASS: recursive conditional type
type DeepReadonly<T> = {
  readonly [K in keyof T]: T[K] extends object
    ? DeepReadonly<T[K]>
    : T[K];
};

// FAIL: without recursion, nested objects remain mutable
type ShallowReadonly<T> = {
  readonly [K in keyof T]: T[K];
};
```

---

## 8. Variadic Tuple Types

### Basic Patterns

```typescript
// PASS: variadic tuple for flexible function arguments
type MixedTuple = [string, ...number[], boolean];

const good1: MixedTuple = ["hello", true];           // OK
const good2: MixedTuple = ["hello", 1, 2, 3, true];  // OK
// FAIL: wrong order
// const bad: MixedTuple = [true, 1, "hello"];
```

### Type-Safe `Promise.all`

```typescript
// PASS: using variadic tuples for Promise.all typing
declare function promiseAll<const T extends readonly unknown[]>(
  promises: T
): {
  [K in keyof T]: T[K] extends Promise<infer U> ? U : never;
};

const result = promiseAll([
  Promise.resolve(42),
  Promise.resolve("hello"),
  Promise.resolve(true),
]);
// type: [number, string, boolean]

// FAIL: without variadic tuple support, result type is (number | string | boolean)[]
// const resultBad = await Promise.all([42, "hello", true]);
```

### Function Overloads with Variadic Tuples

```typescript
// PASS: variadic tuple eliminates need for multiple overloads
function concat<T extends readonly unknown[]>(
  ...args: [...T]
): [...T];

const arr1 = concat([1, 2, 3]);             // number[]
const arr2 = concat([1, "a", true] as const); // readonly [1, "a", true]

// FAIL: without variadic tuples, you'd write N overloads
// function concat1(a: number[]): number[];
// function concat2(a: number[], b: number[]): number[];
// ...ad infinitum
```

### Partial Parameter Application

```typescript
// PASS: variadic tuples for function composition
type Tail<T extends readonly unknown[]> = T extends [unknown, ...infer Rest] ? Rest : [];

function first<A, Rest extends unknown[]>(
  fn: (...args: Rest) => A,
  ...args: Rest
): () => A {
  return () => fn(...args);
}
```

---

## 9. Advanced Generics

### Constrained Generics

```typescript
// PASS: constrained generic with extends
interface HasId {
  id: string;
}

function getById<T extends HasId>(items: T[], id: string): T | undefined {
  return items.find(item => item.id === id);
}

// FAIL: calling with incompatible type
// function bad(items: number[], id: string) {
//   return getById(items, id); // number doesn't extend HasId
// }
```

### Default Type Parameters

```typescript
// PASS: generic with default type
function createStore<T = string>(initial: T) {
  let state = initial;
  return {
    get: () => state,
    set: (next: T) => { state = next; },
  };
}

const store1 = createStore(42);    // Store<number>
const store2 = createStore();       // Store<string> — uses default
```

### `NoInfer<T>` (TypeScript 5.4+)

```typescript
// PASS: NoInfer prevents TypeScript from using a position for inference
function create<T extends string>(
  items: T[],
  // comparator doesn't contribute to T inference
  comparator?: (a: NoInfer<T>, b: NoInfer<T>) => number,
): T[] {
  return items.sort(comparator);
}

const sorted = create(["b", "a", "c"], (a, b) => a.localeCompare(b));

// FAIL: without NoInfer, comparator parameter would widen T to string
// function createBad<T extends string>(items: T[], comparator?: (a: T, b: T) => number): T[] {
//   return items.sort(comparator);
// }
// const sortedBad = createBad(["b", "a", "c"], (a, b) => a.localeCompare(b));
```

### Const Type Parameters (TS 5.0+)

```typescript
// PASS: const type parameter preserves literal types
function tuple<const T extends readonly string[]>(...args: T): T {
  return args;
}

const t = tuple("a", "b", "c");
// type: readonly ["a", "b", "c"] — not string[]

// FAIL: without const, literals widen
function tupleBad<T extends string[]>(...args: T): T {
  return args;
}
const tBad = tupleBad("a", "b", "c");
// type: string[] — lost the literal values
```

---

## 10. Utility Types Avançados

### Built-in Utilities

```typescript
// PASS: Pick — select specific keys
interface User {
  id: string;
  name: string;
  email: string;
  password: string;
  role: "admin" | "user";
}

type PublicUser = Pick<User, "id" | "name" | "email">;

// PASS: Omit — exclude specific keys
type UserWithoutPassword = Omit<User, "password">;

// PASS: Exclude and Extract for union types
type Status = "pending" | "active" | "inactive" | "deleted";
type ActiveStatus = Exclude<Status, "deleted" | "inactive">; // "pending" | "active"
type DeletedStatus = Extract<Status, "deleted">;              // "deleted"

// PASS: NonNullable
type Maybe = string | null | undefined;
type Definitely = NonNullable<Maybe>; // string

// PASS: Partial, Required, Readonly
type PartialUser = Partial<User>;
type RequiredUser = Required<PartialUser>;
type ReadonlyUser = Readonly<User>;
```

### Record Pattern

```typescript
// PASS: Record<K,V> for dictionary types
type PageRoutes = Record<string, string>;
type StatusCodes = Record<number, string>;

// PASS: Record with union keys
type UserRole = "admin" | "editor" | "viewer";
type RolePermissions = Record<UserRole, string[]>;

const permissions: RolePermissions = {
  admin: ["read", "write", "delete"],
  editor: ["read", "write"],
  viewer: ["read"],
};

// FAIL: missing a key would error
// const badPermissions: RolePermissions = {
//   admin: ["read"],
//   editor: ["read", "write"],
// }; // missing "viewer"
```

### Custom PickByValue / OmitByValue

```typescript
// PASS: PickByValue using conditional mapped types
type PickByValue<T, V> = {
  [K in keyof T as T[K] extends V ? K : never]: T[K];
};

interface Example {
  id: string;
  age: number;
  name: string;
  active: boolean;
}

type StringKeys = PickByValue<Example, string>;
// { id: string; name: string }

// PASS: OmitByValue
type OmitByValue<T, V> = {
  [K in keyof T as T[K] extends V ? never : K]: T[K];
};

type WithoutStrings = OmitByValue<Example, string>;
// { age: number; active: boolean }
```

---

## 11. Mapped Types

### Basic Mapped Type

```typescript
// PASS: mapped type over object keys
type Nullable<T> = {
  [K in keyof T]: T[K] | null;
};

interface Config {
  host: string;
  port: number;
  debug: boolean;
}

type NullableConfig = Nullable<Config>;
// { host: string | null; port: number | null; debug: boolean | null }

// FAIL: manual version repeats keys
// type NullableConfigManual = {
//   host: string | null;
//   port: number | null;
//   debug: boolean | null;
// };
```

### DeepPartial and DeepReadonly

```typescript
// PASS: recursive mapped type
type DeepPartial<T> = {
  [K in keyof T]?: T[K] extends object ? DeepPartial<T[K]> : T[K];
};

interface Nested {
  user: {
    profile: {
      name: string;
      age: number;
    };
    settings: {
      theme: "light" | "dark";
    };
  };
}

type DeepPartialNested = DeepPartial<Nested>;
// all nested properties are optional

// PASS: DeepReadonly
type DeepReadonly<T> = {
  readonly [K in keyof T]: T[K] extends object ? DeepReadonly<T[K]> : T[K];
};
```

### Key Remapping with `as` Clause (TS 4.1+)

```typescript
// PASS: key remapping with as clause
type Getters<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K];
};

interface Person {
  name: string;
  age: number;
}

type PersonGetters = Getters<Person>;
// { getName: () => string; getAge: () => number }

// PASS: filtering keys with as never
type MethodsOnly<T> = {
  [K in keyof T as T[K] extends (...args: any) => any ? K : never]: T[K];
};

interface Service {
  name: string;
  start(): void;
  stop(): void;
  version: number;
}

type ServiceMethods = MethodsOnly<Service>;
// { start: () => void; stop: () => void }

// FAIL: without key remapping, you can't rename or filter keys
// type GettersOld<T> = {
//   [K in keyof T]: () => T[K]; // no way to rename "name" to "getName"
// };
```

---

## 12. Zod + TypeScript

### `z.infer<typeof schema>` Pattern

```typescript
import { z } from "zod";

// PASS: infer types from schemas — single source of truth
const UserSchema = z.object({
  id: z.string().uuid(),
  name: z.string().min(1).max(100),
  email: z.string().email(),
  age: z.number().int().positive().optional(),
  role: z.enum(["admin", "user", "viewer"]),
});

type User = z.infer<typeof UserSchema>;
// {
//   id: string;
//   name: string;
//   email: string;
//   age?: number | undefined;
//   role: "admin" | "user" | "viewer";
// }

// PASS: input type for creation (before defaults/transforms)
type UserInput = z.input<typeof UserSchema>;
// -- differs from output when defaults/transforms are used

// FAIL: manually duplicating types drifts
// interface UserManual { /* ... */ }
// const UserSchemaManual = z.object({ /* ... */ });
// // update one but forget the other — mismatch
```

### Zod Discriminated Unions

```typescript
// PASS: zod discriminated union matches TS discriminated union
const SuccessSchema = z.object({
  success: z.literal(true),
  data: z.unknown(),
});

const ErrorSchema = z.object({
  success: z.literal(false),
  error: z.string(),
});

const ApiResultSchema = z.discriminatedUnion("success", [
  SuccessSchema,
  ErrorSchema,
]);

type ApiResult = z.infer<typeof ApiResultSchema>;
// { success: true; data: unknown } | { success: false; error: string }

// PASS: narrowing works after parsing
const parsed = ApiResultSchema.parse(response);
if (parsed.success) {
  console.log(parsed.data); // unknown
} else {
  console.log(parsed.error); // string
}

// FAIL: z.union() won't narrow as cleanly
// const BadUnion = z.union([SuccessSchema, ErrorSchema]);
// // you need a custom discriminator or manual narrow
```

### Branded Zod Types

```typescript
import { z } from "zod";

// PASS: Zod branded types for nominal typing at runtime
const UserIdSchema = z.string().uuid().brand("UserId");
const PostIdSchema = z.string().uuid().brand("PostId");

type UserId = z.infer<typeof UserIdSchema>;
type PostId = z.infer<typeof PostIdSchema>;

function getUserById(id: UserId) {
  // ...
}

const rawId = "550e8400-e29b-41d4-a716-446655440000";
const userId = UserIdSchema.parse(rawId);
const postId = PostIdSchema.parse(rawId);

getUserById(userId); // OK
// getUserById(postId); // FAIL: Type 'PostId' not assignable to 'UserId'

// PASS: Zod branded types survive JSON serialization (they're still strings at runtime)
const json = JSON.stringify(userId); // "550e8400-..."
```

### Transform Patterns

```typescript
import { z } from "zod";

// PASS: transform to convert at the boundary
const DateStringSchema = z.string().datetime().transform((val) => new Date(val));
type DateString = z.infer<typeof DateStringSchema>; // Date
type DateStringInput = z.input<typeof DateStringSchema>; // string

// PASS: pipeline for multi-step transforms
const SlugSchema = z
  .string()
  .min(1)
  .max(200)
  .transform((val) => val.toLowerCase().replace(/\s+/g, "-"))
  .brand("Slug");

type Slug = z.infer<typeof SlugSchema>; // string & { __brand: "Slug" }

// PASS: preprocess for normalizing before validation
const NumberStringSchema = z.preprocess(
  (val) => (typeof val === "string" ? Number(val) : val),
  z.number(),
);

const parsed = NumberStringSchema.parse("42"); // 42

// FAIL: parsing without preprocessing fails on string inputs
// z.number().parse("42"); // ZodError: Expected number, received string
```

## Anti-Patterns to Avoid

### Overusing `any`

```typescript
// FAIL: any disables all type checking
function process(input: any) {
  return input.nonExistent.field; // no error at compile time
}

// PASS: use unknown when you don't know the type
function processSafe(input: unknown) {
  if (typeof input === "object" && input !== null && "field" in input) {
    return (input as { field: unknown }).field;
  }
  throw new Error("Invalid input");
}
```

### Type Assertions as Default

```typescript
// FAIL: as erases type safety
const user = response as User;

// PASS: validate at the boundary
function assertUser(value: unknown): asserts value is User {
  if (typeof value !== "object" || value === null) throw new Error();
  if (typeof (value as User).id !== "string") throw new Error();
}

assertUser(response);
const user: User = response; // safe
```

### Over-Narrowing Return Types

```typescript
// FAIL: too specific — breaks if internals change
function getConfig(): { host: string; port: number } {
  return { host: "localhost", port: 3000 };
}

// PASS: infer return type — let TypeScript figure it out
function getConfig() {
  return { host: "localhost", port: 3000 };
}
// or use satisfies to check shape without losing inference
```

## Recommended Configuration

```jsonc
// tsconfig.json core settings for strict type safety
{
  "compilerOptions": {
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noPropertyAccessFromIndexSignature": true,
    "skipLibCheck": false
  }
}
```
