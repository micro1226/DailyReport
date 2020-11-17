# 横竖屏切换及锁定

## 关键词
--------
**iOS** **横竖屏切换** **横竖屏锁定** **Rotate**


## 内容
--------
### **需求**
项目打开是【登录】页面，登录成功之后进入【选择模块】页面，有【A】、【B】两个模块可供选择。
其中【登录】页面和【选择模块】页面都是支持旋转的,【A】页面只支持竖屏,【B】页面只支持横屏

### **思想**
1. 扩展 `UINavigationController` 类
    - 重写 `shouldAutorotate` 方法: 返回`topViewController!.shouldAutorotate`
    - 重写 `supportedInterfaceOrientations` 方法: 返回`topViewController!.supportedInterfaceOrientations`
    - 重写 `preferredInterfaceOrientationForPresentation` 方法: 返回`topViewController!.preferredInterfaceOrientationForPresentation`


2. 创建公共的类 `BaseController`, 继承自 `UIViewController`
    - 设置 `BaseController` 支持横竖屏, 即
        ```objc 
        isSupportPortraitOnly = true 
        isSupportLandscapeOnly = true
        ```
    - 重写`shouldAutorotate`方法: 
        1. 通过`statusBarOrientation`来判断 app 当前处于横屏还是竖屏状态
        2. 如果只支持横屏,且当前已经是横屏 返回 NO
        3. 如果只支持竖屏,且当前已经是竖屏,返回 NO
        4. 除此以外返回 YES
    - 重写`supportedInterfaceOrientations`方法:
        1. 如果 `isSupportPortraitOnly == true` 则 返回 `UIInterfaceOrientationMaskLandscapeRight`
        2. 如果 `isSupportLandscapeOnly == true` 则 返回 `UIInterfaceOrientationMaskLandscapeRight`
        3. 除此以外返回 `UIInterfaceOrientationMaskAll`
    - 重写`preferredInterfaceOrientationForPresentation`方法
        1. 同`supportedInterfaceOrientations`方法
    - 定义`setupOrientation`方法
        1. 如果 `isSupportPortraitOnly == true` 则 设置切换屏幕到竖屏
        2. 如果 `isSupportLandscapeOnly == true` 则 设置切换屏幕到横屏
    - 在`BaseController`的`viewWillAppear`方法中调用`setupOrientation`方法

3. 创建公共类 `LandscapeViewController` 继承自 `BaseController`
    - 设置 `LandscapeViewController` 只支持横屏, 即
        ```swift
        isSupportPortraitOnly = false
        isSupportLandscapeOnly = true
        ```
4. 创建公共类 `VerticalViewController` 继承自 `BaseController`
    - 设置 `VerticalViewController` 只支持竖屏, 即
        ```swift
        isSupportPortraitOnly = true
        isSupportLandscapeOnly = false
        ```



### **实现**
- **UINavigationController**
    ```swift
    extension UINavigationController {
        open override var shouldAutorotate: Bool {
            return topViewController!.shouldAutorotate
        }
        
        open override var supportedInterfaceOrientations: UIInterfaceOrientationMask {
            return topViewController!.supportedInterfaceOrientations
        }
            
        open override var preferredInterfaceOrientationForPresentation: UIInterfaceOrientation {
            return topViewController!.preferredInterfaceOrientationForPresentation
        }
    }
    ```
- **BaseController**
    ```objc
    - (BOOL)isSupportLandscapeOnly
    {
        return true;
    }

    - (BOOL)isSupportPortraitOnly
    {
        return true;
    }

    - (BOOL)shouldAutorotate {    
        UIInterfaceOrientation orientation = [UIApplication sharedApplication].statusBarOrientation;
        BOOL isLandscapeNow = orientation == UIInterfaceOrientationLandscapeRight;
        BOOL isPortraitNow = orientation == UIInterfaceOrientationPortrait;    

        if (self.isSupportPortraitOnly && isPortraitNow) {
            return NO;
        }
        if (self.isSupportLandscapeOnly && isLandscapeNow) {
            return NO;
        }
        
        return YES;
    }

    - (UIInterfaceOrientationMask)supportedInterfaceOrientations {
        if (self.isSupportLandscapeOnly) {
            return UIInterfaceOrientationMaskLandscapeRight;
        }
        
        if (self.isSupportPortraitOnly) {
            return UIInterfaceOrientationMaskPortrait;
        }
        
        return UIInterfaceOrientationMaskAll;
    }

    - (UIInterfaceOrientation)preferredInterfaceOrientationForPresentation {
        if (self.isSupportLandscapeOnly) {
            return UIInterfaceOrientationLandscapeRight;
        }
        
        if (self.isSupportPortraitOnly) {
            return UIInterfaceOrientationPortrait;
        }
        
        return UIInterfaceOrientationPortrait;
    }

    - (void)setupOrientation {
        if ([self isKindOfClass:[MenuViewController class]] || [self isKindOfClass:[EMSMenuViewController class]]) {
            return;
        }
        
        if (self.isSupportPortraitOnly) {
            [UIDevice.currentDevice setValue:[NSNumber numberWithInt:UIInterfaceOrientationUnknown] forKey:@"orientation"];
            [UIDevice.currentDevice setValue:[NSNumber numberWithInt:UIInterfaceOrientationPortrait] forKey:@"orientation"];
            [[UIApplication sharedApplication] setStatusBarOrientation:UIInterfaceOrientationPortrait];
        }
        
        if (self.isSupportLandscapeOnly) {
            [UIDevice.currentDevice setValue:[NSNumber numberWithInt:UIInterfaceOrientationUnknown] forKey:@"orientation"];
            [UIDevice.currentDevice setValue:[NSNumber numberWithInt:UIInterfaceOrientationLandscapeRight] forKey:@"orientation"];
            [[UIApplication sharedApplication] setStatusBarOrientation:UIInterfaceOrientationLandscapeRight];
        }
    }
    ```     

-  **LandscapeViewController**
    ```swift
    override var isSupportPortraitOnly: Bool {
        get {
            return false
        }
        set {
            super.isSupportPortraitOnly = false
        }
    }

    override var isSupportLandscapeOnly: Bool {
        get {
            return true
        }
        set {
            super.isSupportLandscapeOnly = true
        }
    }
    ```

- **VerticalViewController**
    ```swift
    override var isSupportPortraitOnly: Bool {
        get {
            return false
        }
        set {
            super.isSupportPortraitOnly = false
        }
    }

    override var isSupportLandscapeOnly: Bool {
        get {
            return false
        }
        set {
            super.isSupportLandscapeOnly = false
        }
    }    
    ```