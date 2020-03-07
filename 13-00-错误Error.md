error异常处理

## 错误类型

* 语法错误（编译报错）
* 逻辑错误
* 运行时错误（可能会导致闪退，一般也叫做异常）
* 自定义错误：通过error协议自定义运行时错误：可能会抛出error的函数必须加上throws声明，并且需要try调用函数

```
struct MyError : Error {
    var msg : String
}

func divide(_ num1:Int,_ num2:Int) throws ->Int{
    if num2 == 0 {
        throw MyError(msg:"0不能作为除数")
    }
    return num1 / num2
}

var result = try divide(1, 0)
print(result)
```

打印：

```
Playground execution terminated: An error was thrown and was not caught:
▿ MyError
  - msg : "0不能作为除数"
```

## 处理error

1. 通过do-catch捕捉并处理：

```
enum MyError : Error {
    case illegalArg(String)
    case outOfBounds(Int,Int)
}

func divide(_ num1:Int,_ num2:Int) throws ->Int{
    if num2 == 0 {
        throw MyError.illegalArg("0不能作为除数")
    }
    return num1 / num2
}
func test(){
    print("111")
    do{
        print("222")
        print(try divide(1, 0))
        print("2333")
    }catch let MyError.illegalArg(msg){
        print("参数异常",msg)
    }catch let MyError.outOfBounds(size, index){
        print("下标越界","size=\(size)","index=\(index)")
    }catch{}
    print("333")
    
}
test()
```
打印：
111
222
参数异常 0不能作为除数
333

一旦捕获到异常，try作用域内后面的代码都不会执行，而是直接进入到catch

2. 直接抛给上层函数

```
func test() throws{
    print(try divide(1, 0))
}
```
如果最顶层函数main函数依然没有捕捉error,那么程序将终止。


3. try?  try!调用可能会抛出error的函数，这样就不用去处理error

```
func test(){
   var result = try? divide(20, 0)
}
```
以下a和b是等价的：

```
var a = try?divide(20,0)
var b : Int
do{
	b = try? divide(20, 0)
}catch{
	b = nil
}
```

## rethows

函数本身不会抛出错误，但调用闭包参数抛出错误，那么它会将错误往上抛

## defer

用来定义以任何方式（抛出错误、return）离开代码块前必须要执行的代码：比如，我们打开一个文件，对文件进行操作，操作完成要关闭文件，但是操作文件的过程中可能抛出了错误，导致后面的代码包括关闭文件的操作都没有执行，这个时候我们就可以加上defer语句，将代码延迟到当前作用域结束之前执行，无论以何种方式结束

```
enum MyError : Error {
    case illegalArg(String)
    case outOfBounds(Int,Int)
}

func divide(_ num1:Int,_ num2:Int) throws ->Int{
    if num2 == 0 {
        throw MyError.illegalArg("0不能作为除数")
    }
    return num1 / num2
}
func open(_ filename:String)->Int{
    print("open")
    return 0
}
func close(_ file:Int){
    print("close")
}
func processFile(_ filename:String) throws{
    let file = open(filename)
    defer {
        close(file)
    }
    try divide(20, 0)
}

try processFile("text.txt")
```

打印结果：

```
open
close
Playground execution terminated: An error was thrown and was not caught:
▿ MyError
  - illegalArg : "0不能作为除数"
```

**defer的语句的执行顺序与定义顺序相反，先写的后调用**


