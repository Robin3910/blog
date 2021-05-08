---
title: ts随笔
---
    该笔记主要记录了一些编码过程中比较常见但是又容易模糊的知识点

**原始类型**：`string`, `boolean`, `number`, `null`, `void`, `symbol`, `undefined`, `bigint`

Tip: 编码过程中也有类似的关键字`Boolean`, `Number`, `String`等，这是JS的构造函数，和类型完全不同。



**Bigint**

使用Bigint可以安全地存储和操作大整数，即这个数已经超过了JS构造函数`Number`能够表示的安全整数范围，使用`Bigint`需要使用`ESnext`编译辅助库。

下面两段代码就可以看出区别：

```ts
const max = Number.MAX_SAFE_INTEGER; // 2^53-1
const max1 = max + 1;
const max2 = max + 2;
console.log(max1 === max2); // true
```

```ts
const max = BigInt(Number.MAX_SAFE_INTEGER); // 2^53-1
const max1 = max + 1n;
const max2 = max + 2n;
console.log(max1 === max2); // false
```





**unknown和any**

`unknown`是更安全的`any`类型，`any`类型在赋值之前，可以通过.去获取属性和方法不会抛错，但是`unknown`会。

```ts
let value: any;
value.foo.bar; // ok

let zoo: unknown;
zoo.foo; // error
```









**元组**

元组与数组非常相似，元组可以说是类型更加严格的数组。

元组定义完后可以通过`push`添加元素，但是获取时会报错

```ts
let x: [string, number];
x = ["robin", 10]; // ok
x = [10]; // error

x.push(2);
x[2]; // error
```







**枚举的本质**

```ts
enum Direction{
		up,
		left,
		down,
		right
}
console.log(Direction.up) // 0
console.log(Direction[0]) // up
```

从上述例子中可以看出枚举中实现了正反向同时映射，平常的例子中，一般都是正向映射，比如一个对象，`key => value`；

直接看看枚举编译后的JS代码：

```js
var Direction;
(function (Direction) {
		Direction[Direction['up'] = 0] = 'up';
  	// ...
}) (Direction || (Direction = {}));
```





**实现一个完善的pick函数，用于获取对象中的value**

```TS
interface KVObject {
    [key: string]: any;
}

function pick<T, K extends keyof T>(o: T, names: Array<K>): Array<T[K]>{
    return names.map((n: K)=>{
        return o[n];
    });
}

const obj: KVObject = {
    username: 'robin',
    age: 25
};

console.log(pick(obj, ["username", 'age']));
```



**把接口中的成员变成可选，这个场景十份常见，我们在声明一个接口后，常常在声明变量的时候去利用这个接口，但是在声明变量时，变量内的属性或者方法一时半会又给不到，这时候利用以下这个方法就可以有效兼容**

```TS
interface User {
    username: string;
    id: number;
    token: string;
    avatar: string;
    role: string;
}


type partial<T> = { [K in keyof T]? : T[K] }


const user: partial<User> = {
    id: 1,
};
```





**类型编程：编写一个工具类型将`interface`中函数类型的名称取出来**

```TS
interface Part {
    id: number;
    name: string;
    subParts: Part[];
    updatePart(newName: string): void;
}

type FunctionPropName<T> = {[K in keyof T]: T[K] extends Function ? K: never}[keyof T];
type funcName = FunctionPropName<Part>; // type funcName = 'updatePart'
```





**类型编程：取出下面`interface`中的可选类型（利用空对象进行扩展甄别，网上看到的一种方法，很巧妙）**

```TS
interface People {
    readonly n: number;
    id: string;
    name: string;
    age?: number;
    from?: string;
}

type OptionalKeys<T> = {[K in keyof T]-?: ({} extends {[P in K]: T[P]} ? K : never)}[keyof T];
type r = OptionalKeys<People>; // 'age'|'from'


// 必须属性
type RequiredKeys<T> = {[K in keyof T]-?: ({} extends {[P in K]: T[P]} ? never : K)}[keyof T];
```
