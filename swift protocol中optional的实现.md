#swift protocol中的optional
在swift中protocol的func是可以使用optional来修饰的，但是为了兼容OC需要使用@objc来修饰整个protocol
```

	 @objc protocol XSProtocol {
   		 @objc optional func optionalFunc() ->Void
	    func normalFunc() -> Void
	  }
	对于此protocol我们声明了两个func：一个 optional 一个必须实现的  
```
public class XSObject: XSProtocol{
      var delegate:XSObjectDelegate?
    
    func normalFunc() {
        print("normalFunc")

    }
    //    func optionalFunc() {
//        print("optionalFunc")
//    }

}


```

对于此的调用我们可以这么做：

```
//检测是否实现了optional的func
  if (xsObj.optionalFunc != nil) {
              xsObj.optionalFunc!()
        }
   xsObj.normalFunc()
   
```


