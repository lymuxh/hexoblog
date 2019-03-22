---
title: iphoneX和iOS11适配遇到问题
date: 2017-09-26 18:25:14
tags: [iPhoneX,iOS11] 
---


## 1.底部tabbar 选中效果下移

```self.tabBar.backgroundImage =[UIColor createImageWithColor:RGB(38.0, 50.0, 56.0)]；
self.tabBar.selectionIndicatorImage = [UIColor createImageWithColor:CLSMainGrayColor size:CGSizeMake([UIScreen mainScreen].bounds.size.width/[_tabControllers count], self.tabBar.bounds.size.height)];
```

改为

```UITabBar *appearance = [UITabBar appearance];
    [appearance setBackgroundImage:[UIColor createImageWithColor:RGB(38.0, 50.0, 56.0)]];
    UIImage *selectionIndicatorImage = [UIColor createImageWithColor:CLSMainGrayColor size:CGSizeMake([UIScreen mainScreen].bounds.size.width/[_tabControllers count], self.tabBar.frame.size.height)];
    selectionIndicatorImage = [selectionIndicatorImage resizableImageWithCapInsets:UIEdgeInsetsMake(1, 0, 0, 0)];
    [appearance setSelectionIndicatorImage:selectionIndicatorImage];
    ```
    
<!-- more -->

## 2. iphoneX下面collectionView显示下移


```self.collectionView = [[UICollectionView alloc]initWithFrame:CGRectMake(0, 0, ScreenWidth, ScreenHeight-StatusRect.size.height-44-48) collectionViewLayout:flowLayout];
```

改为


// iPhone X

`#define  LL_iPhoneX (LL_ScreenWidth == 375.f &&LL_ScreenHeight == 812.f ? YES : NO)`

// Status bar height.

`#define  LL_StatusBarHeight      (LL_iPhoneX ? 44.f : 20.f)`

// Navigation bar height.

`#define  LL_NavigationBarHeight  44.f`

// Tabbar height.

`#define  LL_TabbarHeight         (LL_iPhoneX ? (49.f+34.f) : 49.f)`

// Tabbar safe bottom margin.

`#define  LL_TabbarSafeBottomMargin         (LL_iPhoneX ? 34.f : 0.f)`

// Status bar & navigation bar height.

`#define  LL_StatusBarAndNavigationBarHeight  (LL_iPhoneX ? 88.f : 64.f)`

`
 self.collectionView = [[UICollectionView alloc]initWithFrame:CGRectMake(0, 0, LL_ScreenWidth,LL_ScreenHeight-LL_StatusBarAndNavigationBarHeight-LL_TabbarHeight) collectionViewLayout:flowLayout];
`

## 3.UIToolbar 上面UIButton 点击没响应

 ```UIToolbar *bar = [[UIToolbar alloc] initWithFrame:CGRectMake(0, 0, ScreenWidth, 44)];
    UIButton *button = [[UIButton alloc] initWithFrame:CGRectMake(ScreenWidth-60, 7, 50, 30)];
    [button setTitle:@"完成"forState:UIControlStateNormal];
    button.layer.cornerRadius = 4.0;
    button.backgroundColor = CLSMainGreenColor;
    button.titleLabel.font = CFFontLevel3;
    [button addTarget:self action:@selector(finished) forControlEvents:UIControlEventTouchUpInside];
    [bar addSubview:button];
    self.contentField.inputAccessoryView = bar;
```

改为

```UIToolbar *bar = [[UIToolbar alloc] initWithFrame:CGRectMake(0, 0, LL_ScreenWidth, 44)];
    UIBarButtonItem *space = [[UIBarButtonItem alloc]initWithBarButtonSystemItem:UIBarButtonSystemItemFlexibleSpace target:nil action:nil];
    UIBarButtonItem *finish = [[UIBarButtonItem alloc]initWithTitle:@"完成" style:UIBarButtonItemStyleDone target:self action:@selector(finished)];
    [bar setItems:@[space,space,finish]];
    self.contentField.inputAccessoryView = bar;
```

## 4.navigationBar里面自定义titleView上的按钮点击没有效果

`
self.navigationItem.titleView = _titleView;
`

改为

在自定义titleview 里重写 intrinsicContentSize 属性，代码如下：

`
@property(nonatomic, assign) CGSize intrinsicContentSize;
`
然后在使用地方添加

```
self.navigationItem.titleView = _titleView; 
_titleView.intrinsicContentSize=CGSizeMake(200,40);
```
