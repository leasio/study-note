# 课程目标

- TS实战及面试题

# 知识要点

## 面试题

### 1、类型推论 & 可赋值性

a.什么是类型推论？

答：没有给定指定类型时，ts自动标注类型

b.以下代码ts推论出的类型是什么？

```ts
let a = 1024; // number
let b = '1024'; // string
const c = 'apple'; // 'apple'
let d = [true, false, true]; // boolean[]
let e = { name: 'apple'}; // { name: string }
let f = null; // any
```

![image-20220111000156247](assets/image-20220111000156247-164183051795010.png)

可赋值性：数组，布尔，数字，对象，函数，类、字符串，字⾯量类型，满⾜以下任⼀条件时，A类型可以 赋值给B类型。 

（1）A是B的⼦类型 

（2）A是any类型 规则2是规则1的例外

### 2、类型断言

```ts
function formatInput(input: string):string {
     return input.slice(0, 10);
}
function getUserInput(): string | number {
     return 'test';
}
let input = getUserInput();
formatInput(input as string);
formatInput(<string>input);
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



# 补充知识点

