## 模块

V语言是模块化的语言,程序由模块组成.

函数,结构体,常量,枚举等一级元素都要在模块中定义.

### 定义模块

定义模块的关键字是module.

模块对应的就是文件系统的目录,要定义一个新模块,先创建一个模块目录,然后在目录中创建.v的源文件,一个模块目录可以创建多个源文件.

模块目录名一定要跟源文件中第一行定义的模块名一致.

比如模块目录名为mymodule,那么在目录中的源文件的模块定义也要一样,否则无法编译通过.

```v
module mymodule
```

#### 主模块

```v
//主模块
module main

//在主模块中定义主函数,主函数是程序运行的起点
fn main() {
	
}
```

主模块及其依赖模块编译后,会生成对应平台的单一可执行文件.

#### 库模块

```v
//库模块
module mymodule

fn myfn() {

}
```

库模块编译后会生成对应的库文件,文件扩展名为.o.

### 导入模块

导入模块的关键字是import.


```v
import os
import strings
import mymodule.submodule //导入子模块
```

#### 导入模块取别名

```v
import time as t
import mymodule as mymod
```

上面的2种导入模块方式,使用时都需要带上模块名作为前缀.

#### 短名称导入

模块内公共的函数,结构体,常量可以直接导入,使用时不需要模块名前缀.


```v
module main

// 大括号内是从time模块直接导入的公共函数,结构体,常量
import os { user_os }
import time { now, utc, Time }

// import time as t { now, utc, Time } //也可以模块别名和直接导入同时使用,不过很少场景会同时使用
fn main() {
	println(user_os()) // 直接通过函数名调用,不需要模块前缀
	println(now())
	// println(t.now())
	t := Time{}
	println(t)
}
```

#### 循环依赖检查

当前模块导入其他模块后,就形成了模块依赖,模块间不允许循环依赖,编译器在编译时会进行检查.

#### 模块使用检查

如果导入的模块没有被使用,开发模式(v run xxx.v)下,编译器只是警告，仍然继续编译,方便开发调试，而不用去临时注释掉;生产编译模式(v -prod xxx.v)下，编译器会报错，停止编译.

### 使用模块内容

导入模块后,就可以使用模块中的内容.

目前模块的一级元素有6个:

const常量,enum枚举,fn函数/方法,struct结构体,interface接口,type类型.

只有公共(pub)的一级元素才可以被其他模块访问,非公共的元素只能在模块内部使用.

mymodule.v

```v
module mymodule

// pub是函数的访问控制,只有公共的函数才可以被其他模块使用,没有pub的只能在模块内部使用
pub fn say_hi() {
	println('hello from mymodule!')
}
```

 main.v

```v
module main

import mymodule //导入

fn main() {
	mymodule.say_hi() //使用
}
```

### 模块初始化函数

如果在模块中定义了init函数,当模块被导入时,init函数会被自动调用.

同一个模块内只能定义一个init函数,如果在模块中定义了多个init函数,编译会不通过.

mymodule.v

```v
module mymodule

fn init() {
    println('from init')
}

pub fn my_fn() {
    println('from my_fn')
}
```

main.v

```v
module main

import mymodule


fn main() {
    mymodule.my_fn()
}
```

执行结果是:

```
from init
from my_fn
```

###模块搜索路径

当使用import导入模块时,编译器会按以下顺序搜索模块:

1. 当前编译目录.
2. 当前编译目录中的modules子目录.
3. 标准模块目录,即v/vlib.
4. 第三方模块目录VMODULES.如果设置了VMODULES环境变量,则搜索环境变量指向的目录;如果没有设置VMODULES环境变量,则搜索默认目录~/.vmodules.

