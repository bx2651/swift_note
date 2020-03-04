在创建navigationBarItem时，我们可以先扩展以下UIBarButtonItem的方法，将创建item抽离出来，减少代码量，抽取文件如下：

```
import UIKit

extension UIBarButtonItem{
    class func createItem(imageName:String,highLightImageName:String,size:CGSize)->UIBarButtonItem{
        let btn = UIButton()
        btn.setImage(UIImage(named: imageName), for: .normal)
        btn.setImage(UIImage(named: highLightImageName), for: .highlighted)
        btn.frame = CGRect(origin: CGPoint(x: 0, y: 0), size: size)
        
        return UIBarButtonItem(customView: btn)
        
    }
}
```

上面的写法等同于下面的写法，推荐使用下面的写法

```

import UIKit

extension UIBarButtonItem{
    //在extension中对系统的类进行扩展时，不能使用指定初始化器，所以只能使用便捷初始化器
    convenience init(imageName:String,highLightImageName:String,size:CGSize) {
        let btn = UIButton()
        btn.setImage(UIImage(named: imageName), for: .normal)
        btn.setImage(UIImage(named: highLightImageName), for: .highlighted)
        btn.frame = CGRect(origin: CGPoint(x: 0, y: 0), size: size)
  		 //便捷初始化器必须调用指定初始化器
        self.init(customView:btn)
    }
}
```

下面是具体创建item的代码，需要在创建的页面调用该方法

```
    private func setupNavigationBar(){
        
        let size =  CGSize(width: 60, height: 40)
        //设置左侧item
        //        let btn = UIButton()
        //        btn.setImage(UIImage(named: "search"), for: .normal)
        //        btn.sizeToFit()
        //        btn.frame = rect
        //        navigationItem.leftBarButtonItem = UIBarButtonItem(customView: btn)
        
        //设置右侧item
        //        let historyBtn = UIButton()
        //        historyBtn.setImage(UIImage(named: "history"), for: .normal)
        //        historyBtn.sizeToFit()
        //        historyBtn.frame = rect
        
        let historyItem = UIBarButtonItem.createItem(imageName: "history", highLightImageName:"history_highlight", size: size)
        
        
        let searchItem = UIBarButtonItem.createItem(imageName: "search", highLightImageName:"search_highlight", size: size)
        
        
        let qrcodeItem = UIBarButtonItem.createItem(imageName: "qrcode", highLightImageName:"qrcode_highlight", size: size)
        
        navigationItem.rightBarButtonItems = [historyItem,searchItem,qrcodeItem]
        
    }
    
```