block

#clang编译出源码
```
	clang -rewrite-objc xx.m
	之后会转换出 xx.cpp
```

#	源码
```
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        // insert code here...
        void(^xx)(void) = ^(){
            
        };
        xx(); // __NSGlobalBlock__
        NSLog(@"%@",[xx class]);
        
        BlockObject * blkObj = [[BlockObject alloc] init];
        blkObj.blk = ^{
            
        };
        NSLog(@"%@",[blkObj.blk class]); // __NSGlobalBlock__
        
      /*
       
        ((void (*)(id, SEL, void (*)()))(void *)objc_msgSend)((id)blkObj, sel_registerName("setBlk:"), ((void (*)())&__main_block_impl_2((void *)__main_block_func_2, &__main_block_desc_2_DATA, 截获的变量)));
       
       */
        int a = 10;// 值传递作为参数 直接传点 a
        blkObj.blk = ^{
//            a=1;
            NSLog(@"%d",a);
        };//malloc
        
        NSLog(@"%@",[blkObj.blk class]); //__NSMallocBlock__
        
        static int b = 10;//静态变量的为 &b，引用地址传递
        
        blkObj.blk = ^{
            b =20;
            NSLog(@"%d",b);
        };//global 引用静态
 NSLog(@"%@ %d",[blkObj.blk class], b);
        
        
        __block int c = 10;//(__Block_byref_c_0 *)&c 作为引用传递
        blkObj.blk = ^{
            c = 12;
            
        };// mallocl
        NSLog(@"%@ %d",[blkObj.blk class], c);

        
    }
    return 0;
}

```
编译之后的核心

```	
	
	int main(int argc, const char * argv[]) {
    /* @autoreleasepool */ { __AtAutoreleasePool __autoreleasepool; 

        void(*xx)(void) = ((void (*)())&__main_block_impl_0((void *)__main_block_func_0, &__main_block_desc_0_DATA));
        ((void (*)(__block_impl *))((__block_impl *)xx)->FuncPtr)((__block_impl *)xx);
        NSLog((NSString *)&__NSConstantStringImpl__var_folders_07_g9d89v7j3cnfl9qb1klszxzr0000gn_T_main_5a37ea_mi_0,((Class (*)(id, SEL))(void *)objc_msgSend)((id)xx, sel_registerName("class")));

        BlockObject * blkObj = ((BlockObject *(*)(id, SEL))(void *)objc_msgSend)((id)((BlockObject *(*)(id, SEL))(void *)objc_msgSend)((id)objc_getClass("BlockObject"), sel_registerName("alloc")), sel_registerName("init"));
        ((void (*)(id, SEL, void (*)()))(void *)objc_msgSend)((id)blkObj, sel_registerName("setBlk:"), ((void (*)())&__main_block_impl_1((void *)__main_block_func_1, &__main_block_desc_1_DATA)));
        NSLog((NSString *)&__NSConstantStringImpl__var_folders_07_g9d89v7j3cnfl9qb1klszxzr0000gn_T_main_5a37ea_mi_1,((Class (*)(id, SEL))(void *)objc_msgSend)((id)((void (*(*)(id, SEL))())(void *)objc_msgSend)((id)blkObj, sel_registerName("blk")), sel_registerName("class")));


        int a = 10;
        ((void (*)(id, SEL, void (*)()))(void *)objc_msgSend)((id)blkObj, sel_registerName("setBlk:"), ((void (*)())&__main_block_impl_2((void *)__main_block_func_2, &__main_block_desc_2_DATA, a)));// a的值传入作为参数

        NSLog((NSString *)&__NSConstantStringImpl__var_folders_07_g9d89v7j3cnfl9qb1klszxzr0000gn_T_main_5a37ea_mi_3,((Class (*)(id, SEL))(void *)objc_msgSend)((id)((void (*(*)(id, SEL))())(void *)objc_msgSend)((id)blkObj, sel_registerName("blk")), sel_registerName("class")));

        static int b = 10;

        ((void (*)(id, SEL, void (*)()))(void *)objc_msgSend)((id)blkObj, sel_registerName("setBlk:"), ((void (*)())&__main_block_impl_3((void *)__main_block_func_3, &__main_block_desc_3_DATA, &b)));// 传递静态变量的地址
 NSLog((NSString *)&__NSConstantStringImpl__var_folders_07_g9d89v7j3cnfl9qb1klszxzr0000gn_T_main_5a37ea_mi_5,((Class (*)(id, SEL))(void *)objc_msgSend)((id)((void (*(*)(id, SEL))())(void *)objc_msgSend)((id)blkObj, sel_registerName("blk")), sel_registerName("class")), b);

        __attribute__((__blocks__(byref))) __Block_byref_c_0 c = {(void*)0,(__Block_byref_c_0 *)&c, 0, sizeof(__Block_byref_c_0), 10};
        ((void (*)(id, SEL, void (*)()))(void *)objc_msgSend)((id)blkObj, sel_registerName("setBlk:"), ((void (*)())&__main_block_impl_4((void *)__main_block_func_4, &__main_block_desc_4_DATA, (__Block_byref_c_0 *)&c, 570425344)));//__block 的转化之后也是用 地址最为参数
        NSLog((NSString *)&__NSConstantStringImpl__var_folders_07_g9d89v7j3cnfl9qb1klszxzr0000gn_T_main_5a37ea_mi_6,((Class (*)(id, SEL))(void *)objc_msgSend)((id)((void (*(*)(id, SEL))())(void *)objc_msgSend)((id)blkObj, sel_registerName("blk")), sel_registerName("class")), (c.__forwarding->c));


    }
    return 0;
}
	
```
