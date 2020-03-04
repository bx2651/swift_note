自定义log的实现

```
func myLog<T>(_ message:T, file:String = #file, function:String = #function,
           line:Int = #line) {
    #if DEBUG
        //获取文件名
        let fileName = (file as NSString).lastPathComponent
        //打印日志内容
        print("\(fileName):\(line) \(function) | \(message)")
    #endif
}

myLog("Home log")
```

打印效果
```
MainViewController.swift:28 homeButton(_:) | Home log
```