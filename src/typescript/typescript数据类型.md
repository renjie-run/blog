## 基础静态类型
基础静态类型有数字、字符串、布尔值等，具体定义方式如下
```ts
// 数字
const count: number = 1;

// 字符串
const name: string = 'Jerry';

// 布尔值
const checked: boolean = true;

// 空：void
const res: void = undefined;
const func = (): viod => {
  // without return
}

// undefined / null
const un: undefined = undefined; // const un: undefined = null
const nul: null = null; // const nul: null = undefined;

// 任意类型：any
const res2: any = 1; 可以赋任意类型的值，非特殊情况一般不使用

const res3: object = {}; // const res3: object = []
```

### never
一般用在不可能返回内容的函数的返回值类型。例如抛出异常。
```ts
function err(): never {
  throw Error('error')
}
```

## 对象类型
```ts
const obj: {
  name: string,
  age: number
} = {
  name: 'Jerry',
  age: 18,
}
```

### 数组类型
```ts
const obj2: String[] = ['a', 'b'];
```

### 类类型
```ts
class Animal {};

const cat: Animal = new Animal(); // 要求 cat 必须是 Animal 类对应的对象
```

### 函数类型
```ts
const func: () => string = () => {
  return 'hello'
}
```

## 元组（Tuple）
严格限定数字中每个元素的数据类型。

```ts
const tup: [number, string] = [1, 'a']
const tup2: [number, string][] = [
  [1, 'a'],
  [2, 'b']
]
```

## 枚举类型：enum
程序中使用枚举可提高程序的可读性。

```ts
enum Agenda {
  female = 1,
  male = 0,
}

// 未指定对应值时，默认从 0 开始
enum Status {
  OFF,
  ON
}
```

## 数组类型
```ts
// 类型推断
let arr = [1, 2, 3] // arr: number[]

// 数字类型数组
let arr1: number[] = [1, 2, 3]

// 字符串类型数组
let arr2: string[] = ['a', 'b']

// 混合数组
let arr3: (number | string)[] = ['a', '2']

// 数组中的对象类型
let arr4: { name: string, age: number }[] = [
  { name: 'Jerry', age: 18 },
  { name: 'Tom', age: 16 }
]

// 也可以使用类型别名（type）或定义一个类来实现
type User = {
  name: string,
  age: number,
}
let arr4: User[] = [
  { name: 'Jerry', age: 18 },
]
```

