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

// 数组对象类型
const obj2: String[] = ['a', 'b'];
```


