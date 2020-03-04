通过MainViewController创建的storyboard:

1. 将main.storyboard的class改为MainViewController

2. 在MainViewController.swift文件创建每个storyboard

```
import UIKit

class MainViewController: UITabBarController {
    override func viewDidLoad() {
        super.viewDidLoad()
        
        addChildVc(storyName: "Home")
        addChildVc(storyName: "Profile")
    }
    
    private func addChildVc(storyName:String){
        //1.通过storyboard获取控制器
        let childVc = UIStoryboard(name: storyName, bundle: nil).instantiateInitialViewController()!
        //2.将childVc作为子控制器
        addChild(childVc)
    }
}

```