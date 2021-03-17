
#### interface
定义object，动态给object添加属性
```javascript
interface IsObject {
    [key: string]: object;
}


##### ? 可选运算

```javascript
function onClick(callback?: () => void) {
    callback!();  // 参数是可选入参，加了这个感叹号!之后，TS编译不报错
  }
  onClick()
```

##### ! 非空断言操作符

`!` 可以用于断言操作对象是非 null 和非 undefined 类型

```javascript
type NumGenerator = () => number;
function myFunc(numGenerator: NumGenerator | undefined) {
  const num1 = numGenerator(); // Error
  const num2 = numGenerator!(); //OK
}
myFunc(undefined)

```

##### ?? 空值合并运算符

`??`在左侧表达式结果为 null 或者 undefined 时，才会返回右侧表达式。

```javascript
let b = a ?? 10
```

##### unknown

在静态编译的时候，unknown 不能调用任何方法，而 any 可以

```javascript
const foo: unknown = 'string'
foo.substr(1) // Error: 静态检查不通过报错
const bar: any = 10
bar.substr(1) // Pass: any类型相当于放弃了静态检查
```

##### never

是指没法正常结束返回的类型，一个必定会报错或者死循环的函数会返回这样的类型

```javascript
function foo(): never {
	throw new Error('error message')
} // throw error 返回值是never
function foo(): never {
	while (true) {}
} // 这个死循环的也会无法正常退出
function foo(): never {
	let count = 1
	while (count) {
		count++
	}
} // Error: 这个无法将返回值定义为never，因为无法在静态编译阶段直接识别出
```

##### keyof 键值获取

keyof 可以获取一个类型所有键值，返回一个联合类型

```javascript
type Person = {
  name: string;
  age: number;
}
type PersonKey = keyof Person;  // PersonKey得到的类型为 'name' | 'age'
```

##### typeof 实例类型获取

typeof 是获取一个对象/实例的类型,只能用在具体的对象上

```javascript
const me: Person = { name: 'gzx', age: 16 }
type P = typeof me // { name: string, age: number | undefined }
const you: typeof me = { name: 'mabaoguo', age: 69 } // 可以通过编译

type PersonKey = keyof typeof me;   // 'name' | 'age'
```

##### in 遍历属性

in 只能用在类型的定义中，可以对枚举类型进行遍历

```javascript
// 这个类型可以将任何类型的键值转化成number类型，外围用[]包裹起来(这个是固定搭配)，冒号右侧number将所有的key定义为number类型
type TypeToNumber<T> = {
  [key in keyof T]: number
}
```

##### 泛型基本适用

```javascript
// 普通类型定义
type Dog<T> = { name: string, type: T }
// 普通类型使用
const dog: Dog<number> = { name: 'ww', type: 20 }

// 类定义
class Cat<T> {
  private type: T;
  constructor(type: T) { this.type = type; }
}
// 类使用
const cat: Cat<number> = new Cat<number>(20); // 或简写 const cat = new Cat(20)

// 函数定义
function swipe<T, U>(value: [T, U]): [U, T] {
  return [value[1], value[0]];
}
// 函数使用
swipe<Cat<number>, Dog<number>>([cat, dog])  // 或简写 swipe([cat, dog])
```

##### 泛型推导与默认值

```javascript
type Dog<T> = { name: string, type: T }

function adopt<T>(dog: Dog<T>) {
	return dog
}

const dog = { name: 'ww', type: 'hsq' } // 这里按照Dog类型的定义一个type为string的对象
adopt(dog) // Pass: 函数会根据入参类型推断出type为string

// 若不适用函数泛型推导，我们若需要定义变量类型则必须指定泛型类型
const dog: Dog<string> = { name: 'ww', type: 'hsq' } // 不可省略<string>这部分
```

如果我们想不指定，可以使用泛型默认值的方案。

```javascript
type Dog<T = any> = { name: string, type: T }
const dog: Dog = { name: 'ww', type: 'hsq' }
dog.type = 123 // 不过这样type类型就是any了，无法自动推导出来，失去了泛型的意义
```

##### 泛型约束

```javascript
function sum<T extends number>(value: T[]): number {
  let count = 0;
  value.forEach(v => count += v);
  return count;
}
sum([1,2,3])//这种方式调用求和函数，
sum(['1', '2'])//这种是无法通过编译的。
```

泛型约束也可以用在多个泛型参数的情况

```javascript
function pick<T, U extends keyof T>(){};
// 这里的意思是限制了 U 一定是 T 的 key 类型中的子集，这种用法常常出现在一些泛型工具库中
```

##### 泛型条件

```javascript
T extends U? X: Y
// 这里便不限制 T 一定要是 U 的子类型，如果是 U 子类型，则将 T 定义为 X 类型，否则定义为 Y 类型
```

##### 泛型推断 infer

infer 的中文是“推断”的意思，一般是搭配上面的泛型条件语句使用的，所谓推断，就是你不用预先指定在泛型列表中，在运行时会自动判断，不过你得先预定义好整体的结构

```javascript
type Foo<T> = T extends {t: infer Test} ? Test: string
// 首选看 extends 后面的内容，{t: infer Test}可以看成是一个包含t属性的类型定义，这个t属性的 value 类型通过infer进行推断后会赋值给Test类型，如果泛型实际参数符合{t: infer Test}的定义那么返回的就是Test类型，否则默认给缺省的string类型

type One = Foo<number>  // string，因为number不是一个包含t的对象类型
type Two = Foo<{t: boolean}>  // boolean，因为泛型参数匹配上了，使用了infer对应的type
type Three = Foo<{a: number, t: () => void}> // () => void，泛型定义是参数的子集，同样适配
let one:One = 'a'
let two:Two = false
const three: Three = function () { }
```

##### 泛型工具 Partical<T>属性全部可选

```javascript
type Partial<T> = {
 [P in keyof T]?: T[P]
}
// 如：
type Animal = {
  name: string,
  category: string,
  age: number,
  eat: () => number
}
type PartOfAnimal = Partical<Animal>;
const ww: PartOfAnimal = { name: 'ww' }; // 属性全部可选后，可以只赋值部分属性了
```

##### 泛型工具 Required<T>属性全部变为必选项。

```javascript
type Required<T> = {
  [P in keyof T]-?: T[P]  //-?，你可以理解为就是 TS 中把?可选属性减去的意思
}
// 如：
type Animal = {
  name?: string,
  category?: string,

}
type PartOfAnimal = Required<Animal>;
const ww: PartOfAnimal = { name: 'ww' ,category:'345'};
```

##### 泛型工具 Record<K, T>

此工具的作用是将 K 中所有属性值转化为 T 类型，我们常用它来申明一个普通 object 对象。

```javascript
// 这里特别说明一下，keyof any对应的类型为number | string | symbol
type Record<K extends keyof any,T> = {
  [key in K]: T
}
// 如：
const obj: Record<string, string> = { 'name': 'mbg', 'tag': '年轻人不讲武德' }

enum IHttpMethods {
  GET = 'get',
  POST = 'post',
}
interface IHttpFn {
  yy: string,
  zz: number
}
type IHttp = Record<IHttpMethods, IHttpFn>;
let obj: IHttp = {
  get: {
    yy: 'string',
    zz: 12
  },
  post: {
    yy: 'string',
    zz: 12
  }
}
```

##### 泛型工具 Pick<T, K> ---提取公共属性

此工具的作用是将 T 类型中的 K 键列表提取出来，生成新的子键值对类型。

```javascript
type Pick<T, K extends keyof T> = {
  [P in K]: T[P]
}

type Animal = {
  name: string,
  age: number,
  hobby: string
}

const bird: Pick<Animal, "name" | "age"> = { name: 'bird', age: 1 }

```

##### 泛型工具 Exclude<T, U> --- 剔除公共的属性

此工具是在 T 类型中，去除 T 类型和 U 类型的交集，返回剩余的部分。

```javascript
type Exclude<T, U> = T extends U ? never : T

type T1 = Exclude<"a" | "b" | "c", "a" | "b">;   // "c"
type T2 = Exclude<string | number | (() => void), Function>; // string | number
```

##### 泛型工具 Omit<T, K> ---提取删除 k 属性后的

此工具可认为是适用于键值对对象的 Exclude，它会去除类型 T 中包含 K 的键值对。

```javascript
type Omit = Pick<T, Exclude<keyof T, K>>

type Animal = {
  name: string,
  age: number,
  hobby: string,
  eat: () => void
}
const OmitAnimal: Omit<Animal, 'name' | 'age'> = { hobby: 'lion', eat: () => { console.log('eat') } }
```

##### ##### 泛型工具 ReturnType<T>

此工具就是获取 T 类型(函数)对应的返回值类型。

```javascript
function foo(x: string | number): string | number {
	return x
}
type FooType = ReturnType<typeof foo> // string | number
let zz: FooType = 123
