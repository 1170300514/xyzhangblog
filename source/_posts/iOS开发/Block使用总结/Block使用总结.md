---
title: Block使用总结
date: 2020-09-24 15:27:03
tags: 
    - iOS
    - Block
---

### Block应用场景

1. 响应事件

   > 需求：UIViewController中使用UICollectionView构造集合视图，在集合视图中自定义Cell，每个Cell中有一个按钮，需要监听该按钮并对其做出相应

   在CellView中声明Block类型的属性，在按钮点击事件中调用该block

   ```objective-c
   //按钮点击Block
   @property (nonatomic, copy) void (^btnClickedBlock)(void);
   
   // 激活事件
   #pragma mark - 按钮点击事件
   - (IBAction)btnClickedAction:(UIButton *)sender {
       if (self.btnClickedBlock) {
           self.btnClickedBlock();
       }
   }
   ```

   在ViewController中调用UICollectionView生成cell的代理方法`- (UICollectionViewCell *)collectionView:(UICollectionView *)collectionView cellForItemAtIndexPath:(NSIndexPath *)indexPath`时，使用setter方法设置CellView中的block属性，其中包含按钮点击后执行的逻辑。---可以不在定义CellView时确定其执行逻辑，而是通过外部的setter方法确定

   ```objective-c
   // 设置cell 响应事件
   cell.btnClickedBlock = ^(UIButton *sender) {
       // 执行逻辑
   };
   ```

2. 传递数据

   > 在UICollectionView中获取选中Cell的序号，根据选中的Cell执行不同逻辑

   ```objective-c
   @property (strong, nonatomic) void (^handleDidSelectedItem)(int indexPath);
   
   - (void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath
   {
       [tableView deselectRowAtIndexPath:indexPath animated:YES];
       _handleDidSelectedItem ? _handleDidSelectedItem(indexPath) : NULL;
   }
   ```

   

### Block使用注意事项

1. 局部变量与__block修饰符

   block会在所在函数中捕获局部变量，但无法修改局部变量，可以修改全局变量、静态变量。

   - 不能修改局部变量：block捕获的是局部变量的const值，只是名字一样无法修改，如果想修改局部变量可以在定义时使用__block修饰。
   - 可以修改静态变量：静态变量属于类，可以在block中调用

2. 循环引用问题

   Self持有Block对象, 同时在Block内部又调用了Self, 导致了Self持有Block, Block又持有Self，那就会引起循环引用

   - TestCycleRetain.m

     ```objective-c
     - (void) dealloc {
         NSLog(@"no cycle retain");
     } 
     
     - (id) init {
         self = [super init];
         if (self) {
     
             #if TestCycleRetainCase1
             //会循环引用
             self.myblock = ^{
                 [self doSomething];
             };
             #endif NSLog(@"myblock is %@", self.myblock);
         }
         return self;
     } 
     ```
   
   循环引用的避免：声明一个self的弱指针，在block中引用该弱指针
   
   ```objective-c
                   #elif TestCycleRetainCase3
           //不会循环引用
           __weak TestCycleRetain * weakSelf = self;
           self.myblock = ^{
               [weakSelf doSomething];
           };
   ```
   
   上述使用block中存在一个隐患，不知道self什么时候会被释放，为了保证在block中不会被释放，需要配合strong来使用
   
   ```objective-c
__weak __typeof(self) weakSelf = self; 
   self.testBlock =  ^{
       __strong __typeof(weakSelf) strongSelf = weakSelf;
          [strongSelf test]; 
   });
   ```
   
   > From: Raven
   >
   > 以上操作带来一个疑问:**通过弱引用接触循环引用的Self, 如果在Block内部再强引用, 岂不是又循环引用了吗?到底是如何延长Self的生命周期的?**
   >
> **解答**:
   >
> block被赋值到`self`的成员变量时, arc下系统会将该block从栈区拷贝到堆区, 此时block会捕获它所用到的变量使用`__weak`可以保证block和`self`之间不会发生循环引用, 而在Block的代码块内声明`_ _strong`指针, 将会对self产生一个强引用延迟self的释放, 但和直接使用`self`所不同的是, **在Block内声明的指针会在Block的作用域外销毁**, 因此这个循环引用只会持续到block执行完毕为止, 当block执行完毕, strongSelf被销毁, `self`不再被Block所持有, 就可以顺利释放了.
   
   
   
3. 并不是所有Block里面的self都需要weak

   - 调用系统方法：

     ```objective-c
     [UIView animateWithDuration:0.5 animations:^{
             NSLog(@"%@", self);
     }];
     ```

     这个block本身存在于静态方法中，虽然block对self强引用，但self不持有该静态方法，所以完全可以在block内部使用self

### block与内存管理

根据Block在内存中的位置分为三种类型：

- NSGlobalBlock：全局区block，设置在程序的数据区域中
- NSStackBlock：栈区，超出变量作用域则栈上的Block和__block变量都被销毁
- NSMallocBlock：堆区，在变量作用域结束时不受影响

1. 位于全局区：GlobalBlock

   生成在全局区的block的情况：

   - 直接将block定义为全局变量

     ```objective-c
     void(^block)(void) = ^ { NSLog(@"Global Block");};
     int main() {
      
     }
     ```

2. 位于栈内存：StackBlock --最常见的block

   ```objective-c
   NSInteger i = 10; 
   block = ^{ 
        NSLog(@"%ld", i); 
   };
   block;
   ```

3. 位于堆内存：MallocBlock

   堆中block无法直接创建，需要执行copy后才能放入堆中


