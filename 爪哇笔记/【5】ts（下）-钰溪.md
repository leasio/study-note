# 课程目标

- TS实战及面试题

# 知识要点

## 面试题

### 1、类型推论 & 可赋值性

**Q：什么是类型推论？**

A：没有给定指定类型时，ts自动标注类型

**Q：以下代码ts推论出的类型是什么？**

```ts
let a = 1024; // number
let b = '1024'; // string
const c = 'apple'; // 'apple'
let d = [true, false, true]; // boolean[]
let e = { name: 'apple'}; // { name: string }
let f = null; // any
```

**Q：以下是否可互相赋值？**

```ts
let aa = 1024;
let aa1: number = 1024;
aa = aa1; // 可
aa1 = aa; // 可

let bb: string = 'bb';
let bb1: string | number = 'bb1';
bb = bb1; // 可
bb1 = bb; // 可
bb1 = 2; // 可
bb = bb1; // 不可

let i: 3 = 3;
i = 4; // 不可

let j = [1,2,3];
j.push(4); // 可
j.push('5'); // 不可
```

![image-20220111000156247](assets/image-20220111000156247-164183051795010.png)

可赋值性：数组，布尔，数字，对象，函数，类、字符串，字⾯量类型，满⾜以下任⼀条件时，A类型可以赋值给B类型

（1）A是B的⼦类型 

（2）A是any类型 

规则2是规则1的例外

### 2、类型断言

手动指定一个值的类型，可能出现运行时的错误

```ts
function formatInput(input: string): string {
     return input.slice(0, 10);
}
function getUserInput(): string | number {
     return 'test';
}
let input = getUserInput();
formatInput(input as string); // 方法一：as
formatInput(<string>input); // 方法二：<>
```

### 3、type 和 interface 的异同

interface侧重于描述数据结构，type（类型别名）侧重于描述类型

```ts
type age = number;
type dataType = number | string ;
type Method = 'GET' | 'POST' | 'PUT' | 'DELETE';
type User = {
     name: string
     age: number
}
interface User1 extends User {
     age: number;
}
const user1:User1 = {
     name: 'John',
     age: 12
}
```

#### 相同点

a.都可以描述一个对象或者函数

```ts
// interface
interface User {
     name: string;
     age: number;
}
interface SetUser {
     (name: string, age: number): void;
}

// type
type User = {
     name: string;
     age: number;
}
type SetUser = (name: string, age: number): void;
```

b.interface和type都可以拓展，interface可以extends type，type也可以extends interface，效果差不多，语法不同

```ts
// interface extends interface
interface Name {
     name: string;
}
interface User extends Name {
     age: number;
}

// type extends type
type Name = {
     name: string;
}
type User = Name & { age: number }

// interface extends type
type Name = {
     name: string;
}
interface User extends Name {
     age: number;
}

// type extends interface
interface Name {
     name: string;
}
type User = Name & {
     age: number;
}
```

#### 不同点

a.type可以⽤于其它类型 （联合类型、元组类型、基本类型（原始值）），interface不⽀持

```ts
type PartialPointX = { x: number };
type PartialPointY = { y: number };
// union(联合)
type PartialPoint = PartialPointX | PartialPointY;
// tuple(元祖)
type Data = [PartialPointX, PartialPointY];
// primitive(原始值)
type Name = Number;
// typeof的返回值
let div = document.createElement('div');
type B = typeof div;
```

b.interface可以多次定义并被视为合并所有声明成员，type不⽀持

```ts
interface Point {
     x: number;
}
interface Point {
     y: number;
}
const point: Point = { x: 1, y: 2 };

interface User {
     name: string;
     age: number;
}
interface User {
     sex: string;
}
// User接⼝为：
{
     name: string;
     age: number;
     sex: string;
}
```

c.type能使⽤**in**关键字⽣成映射类型，但interface不⾏

```ts
type Keys = 'firstname' | 'surname';
type DudeType = {
     [key in Keys]: string;
};
const test: DudeType = {
     firstname: 'Pawel',
     surname: 'Grzybek',
};
```
**Q：以下哪些会报错？**

```ts
type Options = {
     baseURL: string
     cacheSize?: number
     env?: 'prod' | 'dev'
}
class API {
     constructor(options: Options){}
}


new API({
     baseURL: 'http://myapi.site.com',
     env: 'prod'
}) // 编译通过

new API({
     baseURL: 'http://myapi.site.com',
     badEnv: 'prod'
}) // 编译报错：badEnv是多余属性

new API({
     baseURL: 'http://myapi.site.com',
     badEnv: 'prod'
} as Options) // 编译通过（断言方法）

let badOptions = {
     baseURL: 'http://myapi.site.com',
     badEnv: 'prod'
}
new API(badOptions) // 编译通过：把对象赋值给了⼀个变量之后，ts 就不会做“多余属性”的检查了

let options: Options = {
     baseURL: 'http://myapi.site.com',
     badEnv: 'prod'
} // 编译报错：badEnv是多余属性
new API(badOptions)
```

### 4、装饰器问题

|            | 类装饰器     | 方法装饰器                                                   | 访问器装饰器                                                 | 方法参数装饰器                                               | 属性装饰器                                                   |
| ---------- | :----------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 装饰器参数 | 类的构造函数 | 1、对于静态成员是类的构造函数，对于实例成员是类的原型对象。<br />2、成员的名字。<br />3、成员的属性描述符。 | 1、对于静态成员是类的构造函数，对于实例成员是类的原型对象。<br />2、成员的名字。<br />3、成员的属性描述符。 | 1、对于静态成员是类的构造函数，对于实例成员是类的原型对象。<br />2、参数的名字。<br />3、参数在函数参数列表中的索引。 | 1、对于静态成员是类的构造函数，对于实例成员是类的原型对象。<br />2、成员的名字。 |

执行顺序：

a、有多个参数装饰器时：从最后⼀个参数依次向前执⾏ 

b、⽅法和⽅法参数中参数装饰器先执⾏

c、类装饰器总是最后执⾏

d、⽅法和属性装饰器，谁在前⾯谁先执⾏。因为参数属于⽅法⼀部分，所以参数会⼀直紧紧挨着⽅法执⾏

```ts
// 类装饰器
function Log(target:any) {
    console.log(target);
    console.log("in log decorator");
}

@Log
class A {
    constructor(){
        console.log("constructor")
    }
}
new A();

// 输出：
// [Function: A]
// in log decorator
// constructor
```

```ts
// 方法装饰器
function GET(url: string) {
    console.log('entry GET decorator');
    return function (target:any, methodName: string, descriptor: PropertyDescriptor) {
        console.log('entry GET decorator function');
        !target.$Meta && (target.$Meta = {});
        target.$Meta[methodName] = url;
    }
}

class HelloService {
    constructor() {
        console.log("constructor")
    }
    @GET("xx")
    getUser() {
        console.log('getUser function called');
    }
}
new HelloService().getUser();
console.log((<any>HelloService).$Meta);
console.log((new HelloService() as any).$Meta);

// 输出：
// entry GET decorator
// entry GET decorator function
// constructor
// getUser function called
// undefined
// constructor
// { getUser: 'xx' }
```

```ts
// 方法参数装饰器
function PathParam(paramName: string) {
    return function (target:any, methodName: string, paramIndex: number) {
        !target.$Meta && (target.$Meta = {});
        target.$Meta[paramIndex] = paramName;
    }
}

class HelloService {
    constructor() { }
    getUser( @PathParam("userId") userId: string) { }
}

console.log((<any>HelloService).prototype.$Meta); // {'0':'userId'}
```

```ts
// 定义类装饰器
function logClass(params: string) {
    return function (target: any) {
      console.log(target);
      console.log(params);
    }
  }
  
// 定义属性装饰器
function logProperty(params: any) {
	// target--->类的原型对象；attr--->传入的参数url
    return function (target: any, attr: any) {
        console.log(target, attr);

        target[attr] = params
    }
}
  
@logClass('xxxx')
class HttpClient {
    @logProperty('http://www.baidu.com')
    public url: any | undefined;
    constructor() {

    }
    getUrl() {
        console.log(this.url);
    }
}
  
console.log(new HttpClient().getUrl())

// 输出：
// { getUrl: [Function (anonymous)] } url
// [Function: HttpClient]
// xxxx
// http://www.baidu.com
// undefined
```

### 5、接口类型

属性类接⼝

函数类接⼝ 

可索引接⼝ 

类类型接⼝ 

扩展接⼝

## 实战

### 1、axios 封装

### 2、TS装饰器

# 补充知识点

