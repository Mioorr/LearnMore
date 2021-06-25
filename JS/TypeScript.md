# TypeScript

## 基础类型

一些数据类型（`number`、`string`、`boolean`、`array`、`object`）这里不再赘述，说说特殊的一些。

### Any

表示不希望类型检查器对这些值进行检查，直接让它们通过编译阶段的检查。

### Void

表示没有任何类型。

```typescript
// 当一个函数没有返回值时, 可以设置其返回值类型为void
function warnUser():void {
  console.log("This is my warning message.");
}
```

### Null 和 Undefined

默认情况下，`null`和`undefined`是所有类型的子类型，也就是说`null`和`undefined`可以赋值给其他类型的变量。

但当你指定了`--strictNullChecks`标记，`null`和`undefined`只能赋值给`void`和它们各自。

### Never

表示用不存在的值类型。

```typescript
// 返回never的函数必须存在无法达到的终点
function error(message: string): never {
  throw new Error(message);
}
function infiniteLoop(): never {
  while (true) {
  }
}
```

`never`类型是任何类型的子类型，也可以赋值给任何类型；但没有类型是`never`的子类型，即除它本身之外，没有类型可以赋值给它，即使`any`也不可以。

### 元组 Tuple

表示一个已知元素数量和类型的数组，各个元素的类型不必相同。

```typescript
let x: [string, number];
x = ['hello', 10]; // OK
x = [10, 'hello']; // Error
```

### 枚举 Enum

使用`enum`可以为一组数值赋予友好的名字。

```typescript
enum Color {Red, Green, Blue}
let c: Color = Color.Green;
```

默认情况下从0开始为元素编号。也可以手动的指定成员的数值。

```typescript
// 指定个别成员的数值
enum Color {Red = 1, Green, Blue}
let c: Color = Color.Green;
// 指定全部成员的数值
enum Color {Red = 1, Green = 2, Blue = 3}
let c: Color = Color.Green;
```

枚举类型带来的便利是：可以由枚举的值得到它的名字。

```typescript
enum Color {Red = 1, Green, Blue}
let colorName: string = Color[2];
console.log(colorName); // => Green
```

### 类型断言

类型断言好比其他语言里的类型转换，但不进行特殊的数据检查和解构。

类型断言有2中形式：

1. 尖括号语法

   ```typescript
   let someValue: any = "This is a string";
   let strLength: number = (<string>someValue).length;
   ```

2. `as`语法

   ```typescript
   let someValue: any = "This is a string";
   let strLength: number = (someValue as string).length;
   ```

两种形式是等价的，不过在 TS 里面使用 JSX 时，只有`as`语法是被支持的。

## 变量声明

变量声明的关键字`var`、`let`、`const`，就如同 ES2015 那样使用，这里不赘述。

### 解构

利用解构设置值类型。

**解构用于函数传参**：

```typescript
type C = {a: string, b?: number}; // b后的?表示它是缺省值
function f({a, b}: C): void {
  // ...
}
```

 







