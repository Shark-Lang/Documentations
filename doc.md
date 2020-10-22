# Shark-Lang

## What is Shark
是一个使用GUI结构化编程的一套生态系统, 而不仅仅是一个语言.
里面包含了一个可视化编辑器, 一个输入方法, 一个代码生成, 一个版本管理.
## What do you mean by Structural?
它意味着一种完美的协同编程方式，意味着一种全新的、细粒度的版本控制系统，意味着不仅限于1维的语法语义，
意味着极高的拓展性——这是一种更加现代的编程方式。

### How about vim/emacs/etc users?
反正我的后端就要JSON之类的格式的AST, 牛逼你自己搞一套适配.

### Do you have sort of grammer stuff?
理论上讲, 你写的Canvas插件和IDE插件决定了你的语法, 也就是说你这里显示的可能和别人那里显示的不大一样,但是这个语言不会出现任何的由编译器前端产生的问题.
当你在编写IDE显示出的“富文本”代码时，你在写的是AST。

## Ummm... How do we build it?
你的代码会被保存到一个数据库, 然后后端直接通过数据库编译.

## Package management?
那必然是集成于IDE中的, 当然, 它也必定与部署系统集成.

## Toolchain, Wait, How can you support GIT?
不啊，我们不需要GIT, 因为我们有更现代的, 更细粒度的版本控制系统.
在IDE的编辑过程中, 我们的版本控制系统会将用户所作的每一个操作去冗余后保存下来.

综上所述, 我们的开发——调试——版本控制——集成测试——部署. 由一套工具统一的集成到了IDE中, 这使得开发者的开发体验得到极大的统一.

## FFI STUFF?
我们可能会写一个C语言级别的Structural语言子集或者是使用什么编译选项, 这个子集使用的C的ABI, 所以没有任何问题.

---

# Language Architecture

## Front-end
这个语言将类型系统通过pragma分成无数的Level, 比如在正常的书写过程中, 你不会见到任何关于指针或者引用的细节, 这些将会通过编译器的default pass去做逃逸分析, 除非你打开了`Explicit Memory Management`

### Type System
主要支援几大类型:
```
Primitives:
(I|U)[0-9]+

Hole

Structural:
{(name : Type)*}

Row Poly:
{(name : Types)* | other}

Refinement / Contract:
* expand after type erasure

x : I8 {x > 10 && x < 20}
arr : List<I8> {Sorted}

Update Refinement on Abstraction:
* Hint: use `map` store these, property as key and operation(`+`, `-`) as value, no duplicate keys allowed
sort : (lst : List<T>{Sorted+}) -> Void
insert : (lst : List<T>{Sorted-}, e : T) -> Void
// TODO: 打一顿林子篆让他清醒
foo : (x : Int{Owned-+}) -> Void

refinement type Whatever = (x){x>10 && x<20}
refinement type Whatever2 = (x<e, len>){len>10 && len<20}
a:Int{Whatever}
a:Int{a > 10 && a < 20}
// >, < : Int -> Int -> Int
b:String{Whatever} // compile
foo : (lst : Vec<T, L>{Whatever2}) -> ()

type String
  where Property Console, Network
Int{Sorted} // WTF

Refinement Effects:
去抄 koka，https://github.com/koka-lang/koka，https://thzt.github.io/2019/02/26/algebraic-effects/
send : (s : String) -> Void{Network}
read : () -> File{System}
readLine : (){Console} -> String

let s : String{Mutable} = ""
s = "abc" // fine
let s : String = ""
s = "def" // compile error

内存管理类型:
Label:
x:[:Label]Pointer<Type>

内置指针类型:
GCPTR<T>
GCArena<T>
Arena[:Lebel]<T>
Scope<T>
Ref<T>
RawPtr<T>
```

### Extensions
```
Explicit Memory Management -> show pointer type
Unchecked Refinement -> ignore refinement check

```

## Memory Management
默认情况下使用GC+逃逸分析.
逃逸分析之后堆上类型根据猜测的分配大小使用Scope Ptr或Arena类型.

但是在开启`Explicit Memory Management`之前,这些类型对用户不可见.
在开启`Unsafe Raw Pointers`之前, 不可以使用`RawPtr<T>`.
### LifeTime Allocation Deallocation
Scope = Current Scope, allocate memory when expr is excecuted, free when scope exit.

Arena: a memory pool = Label is the life time, allocate memory when entry label.

GCArena: a memory pool but maneged by GC = allocate when is been used and freed when all the content is not in use, free the entire pool.

GCPTR: traced GC.

Raw: What is lifetime? what is allocation? when to free??

### Move
当然, 在开启手动内存管理前, 是逃逸分析的...所以你不用操心.
检查:
当返回`Scope<T>`的时候, 编译器会报错.
当返回`Arena[:Lable]<T>`的时候,若返回对象的Label非当前Label父类型,报错.
当返回`GC<T>`的时候...GC管理咯~
当返回`Ref<T>`的时候, 是MOVE操作咯~

### GC Algorithm
GC最简单最快速的多线程CMS或者可以选择使用单核的最简单的MS.

## Back-end
LLVM