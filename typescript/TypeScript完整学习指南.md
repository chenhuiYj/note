# TypeScript 完整学习指南

> 面向有 JavaScript 基础的开发者，从入门到精通

## 目录

1. [TypeScript 简介](#typescript-简介)
2. [为什么选择 TypeScript](#为什么选择-typescript)
3. [环境搭建与配置](#环境搭建与配置)
4. [基础类型系统](#基础类型系统)
5. [接口与类型别名](#接口与类型别名)
6. [函数类型](#函数类型)
7. [类与面向对象](#类与面向对象)
8. [泛型](#泛型)
9. [高级类型](#高级类型)
10. [模块系统](#模块系统)
11. [装饰器](#装饰器)
12. [工具类型](#工具类型)
13. [最佳实践](#最佳实践)
14. [从 JavaScript 迁移到 TypeScript](#从-javascript-迁移到-typescript)

---

## TypeScript 简介

TypeScript 是由微软开发的开源编程语言，它是 JavaScript 的超集（superset），这意味着：

- **所有 JavaScript 代码都是有效的 TypeScript 代码**
- TypeScript 添加了**静态类型系统**和其他高级特性
- TypeScript 代码会被编译成 JavaScript 代码运行

### TypeScript vs JavaScript

| 特性 | JavaScript | TypeScript |
|------|-----------|------------|
| 类型检查 | 运行时 | 编译时 |
| 类型系统 | 动态类型 | 静态类型 |
| 编译 | 不需要 | 需要编译 |
| 代码提示 | 有限 | 强大的 IDE 支持 |
| 错误发现 | 运行时 | 编译时 |

---

## 为什么选择 TypeScript

### 1. 类型安全

在编译阶段就能发现潜在的错误，而不是等到运行时：

```typescript
// JavaScript - 运行时才会发现错误
function add(a, b) {
  return a + b;
}
add("1", "2"); // 返回 "12"（字符串拼接）

// TypeScript - 编译时就能发现错误
function add(a: number, b: number): number {
  return a + b;
}
add("1", "2"); // ❌ 编译错误：类型不匹配
```

### 2. 更好的代码提示

IDE 能够根据类型提供准确的代码补全和提示：

```typescript
interface User {
  name: string;
  age: number;
  email: string;
}

const user: User = {
  name: "John",
  age: 30,
  email: "john@example.com"
};

// IDE 会自动提示 user 的属性
user. // 自动提示：name, age, email
```

### 3. 重构更安全

类型系统让重构代码时更加安全，IDE 可以追踪所有使用该类型的地方。

### 4. 文档化代码

类型声明本身就是最好的文档，代码即文档。

---

## 环境搭建与配置

### 安装 TypeScript

#### 全局安装

```bash
npm install -g typescript
```

#### 项目本地安装

```bash
npm install --save-dev typescript
```

### 验证安装

```bash
tsc --version
```

### 初始化 TypeScript 项目

```bash
# 生成 tsconfig.json 配置文件
tsc --init
```

### tsconfig.json 配置说明

```json
{
  "compilerOptions": {
    // 编译目标版本
    "target": "ES2020",
    
    // 模块系统
    "module": "ESNext",
    
    // 模块解析策略
    "moduleResolution": "node",
    
    // 输出目录
    "outDir": "./dist",
    
    // 源代码目录
    "rootDir": "./src",
    
    // 严格模式（推荐开启）
    "strict": true,
    
    // 允许 JavaScript 文件
    "allowJs": true,
    
    // 检查 JavaScript 文件
    "checkJs": false,
    
    // 生成声明文件
    "declaration": true,
    
    // 生成 source map
    "sourceMap": true,
    
    // 移除注释
    "removeComments": false,
    
    // 不生成输出文件（仅检查）
    "noEmit": false,
    
    // 增量编译
    "incremental": true,
    
    // ES 模块互操作
    "esModuleInterop": true,
    
    // 允许合成默认导入
    "allowSyntheticDefaultImports": true,
    
    // 跳过库文件类型检查
    "skipLibCheck": true,
    
    // 强制使用一致的模块解析
    "forceConsistentCasingInFileNames": true,
    
    // 解析 JSON 模块
    "resolveJsonModule": true,
    
    // 隔离模块
    "isolatedModules": true,
    
    // 使用 Node.js 模块解析
    "moduleResolution": "node",
    
    // 启用所有严格类型检查选项
    "strict": true,
    
    // 不允许隐式 any
    "noImplicitAny": true,
    
    // 严格的 null 检查
    "strictNullChecks": true,
    
    // 严格的函数类型检查
    "strictFunctionTypes": true,
    
    // 严格的 bind/call/apply 检查
    "strictBindCallApply": true,
    
    // 严格的属性初始化检查
    "strictPropertyInitialization": true,
    
    // 禁止 this 隐式 any
    "noImplicitThis": true,
    
    // 总是使用严格模式
    "alwaysStrict": true,
    
    // 不允许未使用的局部变量
    "noUnusedLocals": true,
    
    // 不允许未使用的参数
    "noUnusedParameters": true,
    
    // 不允许有 fallthrough 的 case
    "noFallthroughCasesInSwitch": true,
    
    // 路径映射
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    }
  },
  
  // 包含的文件
  "include": [
    "src/**/*"
  ],
  
  // 排除的文件
  "exclude": [
    "node_modules",
    "dist"
  ]
}
```

### 编译 TypeScript

```bash
# 编译单个文件
tsc app.ts

# 编译整个项目（使用 tsconfig.json）
tsc

# 监听模式（自动编译）
tsc --watch
```

---

## 基础类型系统

### 基本类型

TypeScript 支持 JavaScript 的所有基本类型，并提供了额外的类型：

```typescript
// 布尔值
let isDone: boolean = false;

// 数字（支持二进制、八进制、十进制、十六进制）
let decimal: number = 6;
let hex: number = 0xf00d;
let binary: number = 0b1010;
let octal: number = 0o744;

// 字符串
let color: string = "blue";
let fullName: string = `Bob ${color}`; // 模板字符串

// 数组
let list1: number[] = [1, 2, 3];
let list2: Array<number> = [1, 2, 3]; // 泛型语法

// 元组（Tuple）- 固定长度和类型的数组
let tuple: [string, number] = ["hello", 10];
tuple[0].substring(1); // OK
tuple[1].toFixed(2); // OK
// tuple[2] = true; // ❌ 错误：索引超出范围

// 枚举（Enum）
enum Color {
  Red,
  Green,
  Blue
}
let c: Color = Color.Green; // 1

// 可以手动指定枚举值
enum Color2 {
  Red = 1,
  Green = 2,
  Blue = 4
}

// 字符串枚举
enum Direction {
  Up = "UP",
  Down = "DOWN",
  Left = "LEFT",
  Right = "RIGHT"
}

// Any - 任意类型（不推荐使用）
let notSure: any = 4;
notSure = "maybe a string instead";
notSure = false;

// Void - 表示没有任何类型（通常用于函数返回值）
function warnUser(): void {
  console.log("This is my warning message");
}

// Null 和 Undefined
let u: undefined = undefined;
let n: null = null;

// Never - 表示永远不存在的值的类型
function error(message: string): never {
  throw new Error(message);
}

// Object - 表示非原始类型
let obj: object = { x: 0 };
```

### 类型推断

TypeScript 可以自动推断类型，无需显式声明：

```typescript
// TypeScript 自动推断为 number
let x = 3;

// TypeScript 自动推断为 string[]
let arr = ["one", "two", "three"];

// 显式类型声明
let y: number = 3;
```

### 联合类型（Union Types）

表示值可以是多种类型中的一种：

```typescript
let value: string | number;
value = "hello"; // OK
value = 42; // OK
// value = true; // ❌ 错误

// 函数参数
function padLeft(value: string, padding: string | number) {
  if (typeof padding === "number") {
    return Array(padding + 1).join(" ") + value;
  }
  if (typeof padding === "string") {
    return padding + value;
  }
  throw new Error(`Expected string or number, got '${padding}'.`);
}
```

### 字面量类型（Literal Types）

使用具体的值作为类型：

```typescript
// 字符串字面量类型
type Easing = "ease-in" | "ease-out" | "ease-in-out";

function animate(dx: number, dy: number, easing: Easing) {
  // ...
}

animate(0, 0, "ease-in"); // OK
// animate(0, 0, "uneasy"); // ❌ 错误

// 数字字面量类型
type DiceRoll = 1 | 2 | 3 | 4 | 5 | 6;

function rollDice(): DiceRoll {
  return (Math.floor(Math.random() * 6) + 1) as DiceRoll;
}
```

### 类型断言（Type Assertion）

告诉编译器你比它更了解类型：

```typescript
// 方式一：as 语法（推荐）
let someValue: any = "this is a string";
let strLength: number = (someValue as string).length;

// 方式二：尖括号语法（在 JSX 中不可用）
let strLength2: number = (<string>someValue).length;

// 实际应用
interface Cat {
  name: string;
  meow(): void;
}

interface Dog {
  name: string;
  bark(): void;
}

function isCat(animal: Cat | Dog): animal is Cat {
  return (animal as Cat).meow !== undefined;
}

function makeSound(animal: Cat | Dog) {
  if (isCat(animal)) {
    animal.meow(); // TypeScript 知道这里是 Cat
  } else {
    animal.bark(); // TypeScript 知道这里是 Dog
  }
}
```

---

## 接口与类型别名

### 接口（Interface）

定义对象的形状：

```typescript
// 基本接口
interface User {
  name: string;
  age: number;
}

// 可选属性
interface User {
  name: string;
  age: number;
  email?: string; // 可选
}

// 只读属性
interface Point {
  readonly x: number;
  readonly y: number;
}

// 函数类型
interface SearchFunc {
  (source: string, subString: string): boolean;
}

// 可索引类型
interface StringArray {
  [index: number]: string;
}

// 接口继承
interface Animal {
  name: string;
}

interface Dog extends Animal {
  breed: string;
}

// 多个接口继承
interface Shape {
  color: string;
}

interface PenStroke {
  penWidth: number;
}

interface Square extends Shape, PenStroke {
  sideLength: number;
}
```

### 类型别名（Type Alias）

为类型创建新名称：

```typescript
// 基本类型别名
type StringOrNumber = string | number;

// 对象类型
type Point = {
  x: number;
  y: number;
};

// 函数类型
type Greet = (name: string) => string;

// 联合类型
type Status = "pending" | "success" | "error";

// 交叉类型
type Colorful = {
  color: string;
};

type Circle = {
  radius: number;
};

type ColorfulCircle = Colorful & Circle;
```

### 接口 vs 类型别名

| 特性 | Interface | Type Alias |
|------|-----------|------------|
| 扩展 | 使用 `extends` | 使用交叉类型 `&` |
| 合并 | 支持声明合并 | 不支持 |
| 联合类型 | 不支持 | 支持 |
| 元组 | 支持 | 支持 |
| 计算属性 | 不支持 | 支持 |

```typescript
// Interface 声明合并
interface Window {
  title: string;
}

interface Window {
  ts: TypeScriptAPI;
}

// 等同于
interface Window {
  title: string;
  ts: TypeScriptAPI;
}

// Type Alias 不支持声明合并
type Window = {
  title: string;
};

type Window = {
  ts: TypeScriptAPI;
}; // ❌ 错误：重复标识符
```

---

## 函数类型

### 函数声明

```typescript
// 函数声明
function add(x: number, y: number): number {
  return x + y;
}

// 函数表达式
const multiply = function(x: number, y: number): number {
  return x * y;
};

// 箭头函数
const divide = (x: number, y: number): number => {
  return x / y;
};

// 简写形式
const subtract = (x: number, y: number): number => x - y;
```

### 可选参数和默认参数

```typescript
// 可选参数（必须放在必需参数后面）
function buildName(firstName: string, lastName?: string): string {
  if (lastName) {
    return firstName + " " + lastName;
  }
  return firstName;
}

// 默认参数
function buildName2(firstName: string, lastName: string = "Smith"): string {
  return firstName + " " + lastName;
}

// 剩余参数
function buildName3(firstName: string, ...restOfName: string[]): string {
  return firstName + " " + restOfName.join(" ");
}
```

### 函数重载

```typescript
// 函数重载签名
function makeDate(timestamp: number): Date;
function makeDate(m: number, d: number, y: number): Date;

// 函数实现
function makeDate(mOrTimestamp: number, d?: number, y?: number): Date {
  if (d !== undefined && y !== undefined) {
    return new Date(y, mOrTimestamp, d);
  } else {
    return new Date(mOrTimestamp);
  }
}

const d1 = makeDate(12345678);
const d2 = makeDate(5, 5, 5);
// const d3 = makeDate(1, 3); // ❌ 错误：没有匹配的重载
```

### 函数类型

```typescript
// 函数类型注解
let myAdd: (x: number, y: number) => number;

// 函数类型接口
interface MathOperation {
  (x: number, y: number): number;
}

let add: MathOperation = (x, y) => x + y;
```

---

## 类与面向对象

### 基本类

```typescript
class Greeter {
  greeting: string;
  
  constructor(message: string) {
    this.greeting = message;
  }
  
  greet() {
    return "Hello, " + this.greeting;
  }
}

let greeter = new Greeter("world");
```

### 访问修饰符

```typescript
class Animal {
  public name: string; // 公共（默认）
  private age: number; // 私有（只能在类内部访问）
  protected species: string; // 受保护（可以在子类中访问）
  readonly id: number; // 只读
  
  constructor(name: string, age: number, species: string, id: number) {
    this.name = name;
    this.age = age;
    this.species = species;
    this.id = id;
  }
}

// 简写语法（参数属性）
class Animal2 {
  constructor(
    public name: string,
    private age: number,
    protected species: string,
    readonly id: number
  ) {}
}
```

### 继承

```typescript
class Animal {
  name: string;
  
  constructor(name: string) {
    this.name = name;
  }
  
  move(distanceInMeters: number = 0) {
    console.log(`${this.name} moved ${distanceInMeters}m.`);
  }
}

class Dog extends Animal {
  breed: string;
  
  constructor(name: string, breed: string) {
    super(name); // 调用父类构造函数
    this.breed = breed;
  }
  
  move(distanceInMeters: number = 5) {
    console.log("Running...");
    super.move(distanceInMeters); // 调用父类方法
  }
  
  bark() {
    console.log("Woof! Woof!");
  }
}
```

### 抽象类

```typescript
abstract class Animal {
  abstract makeSound(): void; // 抽象方法
  
  move(): void {
    console.log("roaming the earth...");
  }
}

class Dog extends Animal {
  makeSound(): void {
    console.log("Woof! Woof!");
  }
}

// const animal = new Animal(); // ❌ 错误：不能实例化抽象类
const dog = new Dog(); // OK
```

### 实现接口

```typescript
interface ClockInterface {
  currentTime: Date;
  setTime(d: Date): void;
}

class Clock implements ClockInterface {
  currentTime: Date = new Date();
  
  setTime(d: Date): void {
    this.currentTime = d;
  }
  
  constructor(h: number, m: number) {}
}
```

### 静态成员

```typescript
class Grid {
  static origin = { x: 0, y: 0 };
  
  static calculateDistance(point1: { x: number; y: number }, point2: { x: number; y: number }) {
    const xDist = point1.x - point2.x;
    const yDist = point1.y - point2.y;
    return Math.sqrt(xDist * xDist + yDist * yDist);
  }
}

console.log(Grid.origin); // { x: 0, y: 0 }
Grid.calculateDistance({ x: 0, y: 0 }, { x: 3, y: 4 }); // 5
```

### Getter 和 Setter

```typescript
class Employee {
  private _fullName: string = "";
  
  get fullName(): string {
    return this._fullName;
  }
  
  set fullName(newName: string) {
    if (newName && newName.length > 0) {
      this._fullName = newName;
    } else {
      throw new Error("Invalid name");
    }
  }
}

let employee = new Employee();
employee.fullName = "John Doe";
console.log(employee.fullName); // "John Doe"
```

---

## 泛型

### 为什么需要泛型

```typescript
// 不使用泛型 - 需要多个函数
function identityNumber(arg: number): number {
  return arg;
}

function identityString(arg: string): string {
  return arg;
}

// 使用泛型 - 一个函数处理多种类型
function identity<T>(arg: T): T {
  return arg;
}

let output1 = identity<string>("myString");
let output2 = identity<number>(100);
let output3 = identity("myString"); // 类型推断
```

### 泛型函数

```typescript
// 基本泛型函数
function identity<T>(arg: T): T {
  return arg;
}

// 泛型数组函数
function loggingIdentity<T>(arg: T[]): T[] {
  console.log(arg.length);
  return arg;
}

// 使用泛型约束
interface Lengthwise {
  length: number;
}

function loggingIdentity2<T extends Lengthwise>(arg: T): T {
  console.log(arg.length);
  return arg;
}
```

### 泛型接口

```typescript
interface GenericIdentityFn<T> {
  (arg: T): T;
}

function identity<T>(arg: T): T {
  return arg;
}

let myIdentity: GenericIdentityFn<number> = identity;
```

### 泛型类

```typescript
class GenericNumber<T> {
  zeroValue: T;
  add: (x: T, y: T) => T;
}

let myGenericNumber = new GenericNumber<number>();
myGenericNumber.zeroValue = 0;
myGenericNumber.add = function(x, y) {
  return x + y;
};
```

### 泛型约束

```typescript
// 使用 extends 约束泛型
interface Lengthwise {
  length: number;
}

function loggingIdentity<T extends Lengthwise>(arg: T): T {
  console.log(arg.length);
  return arg;
}

// 在泛型约束中使用类型参数
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

let x = { a: 1, b: 2, c: 3, d: 4 };
getProperty(x, "a"); // OK
// getProperty(x, "m"); // ❌ 错误：'m' 不在 'x' 的键中
```

---

## 高级类型

### 交叉类型（Intersection Types）

将多个类型合并为一个类型：

```typescript
interface Colorful {
  color: string;
}

interface Circle {
  radius: number;
}

type ColorfulCircle = Colorful & Circle;

const cc: ColorfulCircle = {
  color: "red",
  radius: 10
};
```

### 联合类型（Union Types）

表示值可以是多种类型中的一种：

```typescript
type Status = "pending" | "success" | "error";

function handleStatus(status: Status) {
  // 类型守卫
  if (status === "pending") {
    // TypeScript 知道这里是 "pending"
  } else if (status === "success") {
    // TypeScript 知道这里是 "success"
  } else {
    // TypeScript 知道这里是 "error"
  }
}
```

### 类型守卫（Type Guards）

缩小类型的范围：

```typescript
// typeof 类型守卫
function padLeft(value: string, padding: string | number) {
  if (typeof padding === "number") {
    return Array(padding + 1).join(" ") + value;
  }
  return padding + value;
}

// instanceof 类型守卫
class Bird {
  fly() {
    console.log("flying");
  }
}

class Fish {
  swim() {
    console.log("swimming");
  }
}

function move(animal: Bird | Fish) {
  if (animal instanceof Bird) {
    animal.fly();
  } else {
    animal.swim();
  }
}

// 自定义类型守卫
function isFish(pet: Fish | Bird): pet is Fish {
  return (pet as Fish).swim !== undefined;
}
```

### keyof 操作符

获取对象所有键的类型：

```typescript
interface Person {
  name: string;
  age: number;
  location: string;
}

type K1 = keyof Person; // "name" | "age" | "location"

function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

let person: Person = {
  name: "John",
  age: 30,
  location: "New York"
};

let name = getProperty(person, "name"); // string
let age = getProperty(person, "age"); // number
```

### typeof 操作符

获取变量的类型：

```typescript
let s = "hello";
let n: typeof s; // string

function f() {
  return { x: 10, y: 3 };
}

type P = ReturnType<typeof f>; // { x: number; y: number }
```

### 索引访问类型（Indexed Access Types）

通过索引访问类型：

```typescript
type Person = {
  age: number;
  name: string;
  alive: boolean;
};

type Age = Person["age"]; // number
type I1 = Person["age" | "name"]; // string | number
type I2 = Person[keyof Person]; // string | number | boolean
```

### 条件类型（Conditional Types）

根据条件选择类型：

```typescript
type IsArray<T> = T extends Array<any> ? true : false;

type A = IsArray<string[]>; // true
type B = IsArray<number>; // false

// 分布式条件类型
type ToArray<T> = T extends any ? T[] : never;

type StrArrOrNumArr = ToArray<string | number>; // string[] | number[]
```

### 映射类型（Mapped Types）

基于旧类型创建新类型：

```typescript
// 将所有属性变为可选
type Partial<T> = {
  [P in keyof T]?: T[P];
};

// 将所有属性变为只读
type Readonly<T> = {
  readonly [P in keyof T]: T[P];
};

// 将所有属性变为必需
type Required<T> = {
  [P in keyof T]-?: T[P];
};

// 移除某些属性
type Omit<T, K extends keyof T> = {
  [P in Exclude<keyof T, K>]: T[P];
};
```

---

## 模块系统

### ES 模块

```typescript
// math.ts
export function add(x: number, y: number): number {
  return x + y;
}

export function subtract(x: number, y: number): number {
  return x - y;
}

// 默认导出
export default function multiply(x: number, y: number): number {
  return x * y;
}

// app.ts
import multiply, { add, subtract } from "./math";
import * as math from "./math";

console.log(add(1, 2));
console.log(math.subtract(5, 3));
```

### 命名空间（Namespace）

```typescript
namespace Validation {
  export interface StringValidator {
    isAcceptable(s: string): boolean;
  }
  
  const lettersRegexp = /^[A-Za-z]+$/;
  const numberRegexp = /^[0-9]+$/;
  
  export class LettersOnlyValidator implements StringValidator {
    isAcceptable(s: string): boolean {
      return lettersRegexp.test(s);
    }
  }
  
  export class ZipCodeValidator implements StringValidator {
    isAcceptable(s: string): boolean {
      return s.length === 5 && numberRegexp.test(s);
    }
  }
}
```

### 声明文件（.d.ts）

为 JavaScript 库提供类型定义：

```typescript
// types.d.ts
declare module "my-module" {
  export function doSomething(): void;
}

// 全局声明
declare global {
  interface Window {
    myCustomProperty: string;
  }
}
```

---

## 装饰器（Decorators）

> 注意：装饰器是实验性特性，需要在 tsconfig.json 中启用 `"experimentalDecorators": true`

### 类装饰器

```typescript
function sealed(constructor: Function) {
  Object.seal(constructor);
  Object.seal(constructor.prototype);
}

@sealed
class BugReport {
  type = "report";
  title: string;
  
  constructor(t: string) {
    this.title = t;
  }
}
```

### 方法装饰器

```typescript
function enumerable(value: boolean) {
  return function(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    descriptor.enumerable = value;
  };
}

class Greeter {
  greeting: string;
  
  constructor(message: string) {
    this.greeting = message;
  }
  
  @enumerable(false)
  greet() {
    return "Hello, " + this.greeting;
  }
}
```

### 属性装饰器

```typescript
function format(formatString: string) {
  return function(target: any, propertyKey: string) {
    // 属性装饰器逻辑
  };
}

class Greeter {
  @format("Hello, %s")
  greeting: string;
}
```

### 参数装饰器

```typescript
function required(target: Object, propertyKey: string | symbol, parameterIndex: number) {
  // 参数装饰器逻辑
}

class Greeter {
  greeting: string;
  
  constructor(@required message: string) {
    this.greeting = message;
  }
}
```

---

## 工具类型

TypeScript 提供了许多内置的工具类型：

### Partial<T>

将所有属性变为可选：

```typescript
interface Todo {
  title: string;
  description: string;
}

function updateTodo(todo: Todo, fieldsToUpdate: Partial<Todo>) {
  return { ...todo, ...fieldsToUpdate };
}

const todo1 = {
  title: "organize desk",
  description: "clear clutter"
};

const todo2 = updateTodo(todo1, {
  description: "throw out trash"
});
```

### Required<T>

将所有属性变为必需：

```typescript
interface Props {
  a?: number;
  b?: string;
}

const obj: Props = { a: 5 }; // OK

const obj2: Required<Props> = { a: 5 }; // ❌ 错误：缺少属性 'b'
```

### Readonly<T>

将所有属性变为只读：

```typescript
interface Todo {
  title: string;
}

const todo: Readonly<Todo> = {
  title: "Delete inactive users"
};

// todo.title = "Hello"; // ❌ 错误：无法分配到 "title"，因为它是只读属性
```

### Record<K, T>

创建对象类型：

```typescript
interface PageInfo {
  title: string;
}

type Page = "home" | "about" | "contact";

const x: Record<Page, PageInfo> = {
  home: { title: "Home" },
  about: { title: "About" },
  contact: { title: "Contact" }
};
```

### Pick<T, K>

选择部分属性：

```typescript
interface Todo {
  title: string;
  description: string;
  completed: boolean;
}

type TodoPreview = Pick<Todo, "title" | "completed">;

const todo: TodoPreview = {
  title: "Clean room",
  completed: false
};
```

### Omit<T, K>

移除部分属性：

```typescript
interface Todo {
  title: string;
  description: string;
  completed: boolean;
}

type TodoPreview = Omit<Todo, "description">;

const todo: TodoPreview = {
  title: "Clean room",
  completed: false
};
```

### Exclude<T, U>

从 T 中排除 U：

```typescript
type T0 = Exclude<"a" | "b" | "c", "a">; // "b" | "c"
type T1 = Exclude<"a" | "b" | "c", "a" | "b">; // "c"
type T2 = Exclude<string | number | (() => void), Function>; // string | number
```

### Extract<T, U>

从 T 中提取 U：

```typescript
type T0 = Extract<"a" | "b" | "c", "a" | "f">; // "a"
type T1 = Extract<string | number | (() => void), Function>; // () => void
```

### NonNullable<T>

排除 null 和 undefined：

```typescript
type T0 = NonNullable<string | number | undefined>; // string | number
type T1 = NonNullable<string[] | null | undefined>; // string[]
```

### ReturnType<T>

获取函数返回类型：

```typescript
type T0 = ReturnType<() => string>; // string
type T1 = ReturnType<(s: string) => void>; // void
type T2 = ReturnType<<T>() => T>; // {}
type T3 = ReturnType<<T extends U, U extends number[]>() => T>; // number[]
```

### Parameters<T>

获取函数参数类型：

```typescript
declare function f1(arg: { a: number; b: string }): void;

type T0 = Parameters<() => string>; // []
type T1 = Parameters<(s: string) => void>; // [string]
type T2 = Parameters<<T>(arg: T) => T>; // [unknown]
type T3 = Parameters<typeof f1>; // [{ a: number; b: string }]
```

---

## 最佳实践

### 1. 优先使用接口而非类型别名（除非需要联合类型）

```typescript
// ✅ 推荐：使用接口
interface User {
  name: string;
  age: number;
}

// ❌ 不推荐：使用类型别名（除非需要联合类型）
type User = {
  name: string;
  age: number;
};
```

### 2. 使用类型推断，避免过度注解

```typescript
// ❌ 过度注解
let x: number = 3;
let arr: number[] = [1, 2, 3];

// ✅ 使用类型推断
let x = 3;
let arr = [1, 2, 3];
```

### 3. 使用 readonly 防止意外修改

```typescript
interface Config {
  readonly apiUrl: string;
  readonly timeout: number;
}
```

### 4. 使用 const 断言

```typescript
// 普通数组
let arr = [1, 2, 3]; // number[]

// const 断言
let arr2 = [1, 2, 3] as const; // readonly [1, 2, 3]
```

### 5. 避免使用 any，使用 unknown

```typescript
// ❌ 不推荐
function process(data: any) {
  return data.someProperty;
}

// ✅ 推荐
function process(data: unknown) {
  if (typeof data === "object" && data !== null && "someProperty" in data) {
    return (data as { someProperty: unknown }).someProperty;
  }
  throw new Error("Invalid data");
}
```

### 6. 使用函数重载而非联合类型参数

```typescript
// ❌ 不推荐
function process(value: string | number): string | number {
  // ...
}

// ✅ 推荐
function process(value: string): string;
function process(value: number): number;
function process(value: string | number): string | number {
  // ...
}
```

### 7. 使用泛型约束而非 any

```typescript
// ❌ 不推荐
function identity(arg: any): any {
  return arg;
}

// ✅ 推荐
function identity<T>(arg: T): T {
  return arg;
}
```

---

## 从 JavaScript 迁移到 TypeScript

### 步骤 1：重命名文件

将 `.js` 文件重命名为 `.ts`（或 `.jsx` 重命名为 `.tsx`）

### 步骤 2：修复类型错误

逐步修复 TypeScript 编译器报告的错误：

```typescript
// 之前（JavaScript）
function add(a, b) {
  return a + b;
}

// 之后（TypeScript）
function add(a: number, b: number): number {
  return a + b;
}
```

### 步骤 3：添加类型注解

为函数参数、返回值、变量等添加类型：

```typescript
// 之前
function fetchUser(id) {
  return fetch(`/api/users/${id}`).then(res => res.json());
}

// 之后
interface User {
  id: number;
  name: string;
  email: string;
}

function fetchUser(id: number): Promise<User> {
  return fetch(`/api/users/${id}`).then(res => res.json());
}
```

### 步骤 4：配置 tsconfig.json

根据项目需求配置 TypeScript 编译器选项。

### 步骤 5：逐步迁移

不需要一次性迁移所有文件，可以：

1. 先迁移核心文件
2. 使用 `// @ts-ignore` 临时忽略错误
3. 逐步完善类型定义

---

## 常见问题与解决方案

### 1. 如何处理第三方库没有类型定义？

```typescript
// 安装 @types 包
npm install --save-dev @types/lodash

// 或者创建声明文件
// types/my-module.d.ts
declare module "my-module" {
  export function doSomething(): void;
}
```

### 2. 如何处理动态属性？

```typescript
// 使用索引签名
interface Dictionary {
  [key: string]: string;
}

// 使用 Record
type Dictionary = Record<string, string>;
```

### 3. 如何处理可选链和空值合并？

```typescript
// TypeScript 3.7+ 支持可选链
let x = foo?.bar?.baz;

// 空值合并
let y = foo ?? "default";
```

### 4. 如何处理 this 类型？

```typescript
class Calculator {
  value = 0;
  
  add(this: Calculator, n: number): this {
    this.value += n;
    return this;
  }
  
  multiply(this: Calculator, n: number): this {
    this.value *= n;
    return this;
  }
}
```

---

## 总结

TypeScript 为 JavaScript 添加了静态类型系统，提供了：

- ✅ **类型安全**：在编译时发现错误
- ✅ **更好的 IDE 支持**：代码补全、导航、重构
- ✅ **代码文档**：类型即文档
- ✅ **重构安全**：类型系统保证重构的正确性

对于有 JavaScript 基础的开发者，学习 TypeScript 的关键是：

1. **理解类型系统**：基本类型、接口、泛型
2. **实践项目**：将现有项目迁移到 TypeScript
3. **利用工具**：充分利用 IDE 的类型提示
4. **逐步学习**：不需要一次性掌握所有特性

开始使用 TypeScript，让代码更安全、更易维护！

---

## 参考资源

- [TypeScript 官方文档](https://www.typescriptlang.org/)
- [TypeScript 中文网](https://www.tslang.cn/)
- [TypeScript Deep Dive](https://basarat.gitbook.io/typescript/)
- [TypeScript 入门教程](https://ts.xcatliu.com/)

---

*本文档持续更新，欢迎补充和完善。*

