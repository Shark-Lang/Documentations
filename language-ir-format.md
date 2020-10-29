
```
// mono
('I32)
// refinement
('I32 {P})
('I32 {+P})
('I32 {-P})
// type function
(=> ([name : Type]) name)
// structural
// I32 == {x:I32} => x == 0
// I32 == {y:I32} => y == 0
(@ [name : Type])
// row
(& type other)
// ↑{x:I32} == {y:I32|} != {x:I32|}
// ↑{x:I32} == {x:I32|} != {y:I32|}
```

```
// module == structrual
// extern
// expr
定义函数 	(:= x : T exp)
绑定 	    (lambda ([x : T]) exp), (letrec ([x : T exp]) exp)
条件判断 	(cond [exp : Bool exp]), (match exp [pat exp])
跳转        (goto label)
赋值 	    (= x : T exp)
```
