---
layout: post
title: "iOS通过RAC实现MVVM"
tagline: "通过RAC可以实现MVVM，当然实现MVVM不一定非得用RAC，只是RAC框架为我们实现MVVM提供了便利。"
---
通过RAC可以实现MVVM，当然实现MVVM不一定非得用RAC，只是RAC框架为我们实现MVVM提供了便利。

下图是我的目录结构，此demo的主要是模拟一个简单的点赞功能。

![1563706536944.jpg](https://iwait.me/assets/imgs/1563706536944.jpg)

其中`TableViewData`类是是用来数据管理，用于数据获取，更新等操作。

```
@interface TableViewData : NSObject
-(instancetype)initWtihUrl:(NSString*)usr;
-(void)lh_load:(LoadBlock)success;//获取数据
-(void)lh_updateLikesAc:(id)model;//点赞操作
@end
```



`MyCell`用来展示数据，`MyCellModel`是用来与Cell关联的model。`MyCell`中绑定代码实现如下：`

```
-(void)lh_cellAssociatedModel:(MyCellModel *)model{
    self.nameLab.text = model.name;
    NSString*imgName = [model.imv isEqualToString:@"0"]?@"dislike":@"like";
    self.imV.image = [UIImage imageNamed:imgName];
    @weakify(self);
    [[RACObserve(model, name) takeUntil:self.rac_prepareForReuseSignal]subscribeNext:^(id  _Nullable x) {
        @strongify(self);
        self.nameLab.text = x;
    }];


    [[RACObserve(model, imv) takeUntil:self.rac_prepareForReuseSignal] subscribeNext:^(id  _Nullable x) {
        @strongify(self);
         NSString* imgName = [x isEqualToString:@"0"]?@"dislike":@"like";
        self.imV.image = [UIImage imageNamed:imgName];
    }];
    }
```



控制器Return cell的代码：

```
-(UITableViewCell*)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath{
    MyCell*cell = [tableView dequeueReusableCellWithIdentifier:@"MyCell"];
    if (cell == nil) {
        cell = [[MyCell alloc]initWithStyle:(UITableViewCellStyleDefault) reuseIdentifier:@"MyCell"];
    }
    [cell lh_cellAssociatedModel:self.dataArray[indexPath.row]];
    @weakify(self);
    [[[cell.clickImvBtn rac_signalForControlEvents:(UIControlEventTouchUpInside)] takeUntil:cell.rac_prepareForReuseSignal]subscribeNext:^(__kindof UIControl * _Nullable x) {
        @strongify(self);
        [self.tdata lh_updateLikesAc:self.dataArray[indexPath.row]];
    }];
    return cell;
}
```

其中用`rac_rac_signalForControlEvents` 使用来监控cell上点赞按钮的状态，监听到点击事件后，通过`TableViewData`实例方法 `lh_updateLikesAc`实现点赞或取消点赞操作。点赞操作成功后会更新model的属性，在model 属性更新后cell会自行进行界面刷新。这样直接在TableViewData 实例中进行数据集中处理，分工明确。尤其是点赞过程中可能遇到的网络问题等，都可以在此进行集中处理。

其中注意的一个点是在cell 被重用的时候原来添加的监听事件该如何处理的问题：

可以通过takeUntil来操作。其中takeUntil操作是监听某个事件直到什么时候结束。rac_prepareForReuseSigna表示当cell即将重用时会触发disposable信号结束监听。
