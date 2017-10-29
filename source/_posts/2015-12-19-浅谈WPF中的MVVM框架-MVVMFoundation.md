title: 浅谈WPF中的MVVM框架--MVVMFoundation
date: 2014-4-19 22:22:25
tags: WPF,MVVM
---
微软对于WPF技术的构想是很宏大的，可惜普及率不高，不过如果你要做Windows客户端开发的话WPF技术还是值得一学的。
<!-- more -->
什么是WPF，请看下图
<img src="http://images.cnitblog.com/blog2015/708791/201504/201140330112092.jpg"  alt="what is WPF" align=center />

#什么是MVVM模式#

简单来说它是一种高级的UI设计模式。据我所知目前还运用在一些js框架中，比如**AngularJS**。其他的UI设计模式还包括MVC、MVP，个人觉得最强大的还是MVVM。

MVVM主体框架如下图：

<img src="http://images.cnitblog.com/blog2015/708791/201504/201225530274012.jpg" width = "500" height = "400" alt="what is MVVM" align=center />


> - The Model is the entity that represents the business concept; it can be anything from a simple customer entity to a complex stock trade entity .
> - The View is the graphical control or set of controls responsible for rendering the Model data on screen .A View can be a WPF window, a Silverlight page, or just an XAML data template control .
> - The ViewModel is the magic behind everything .The ViewModel contains the UI logic, the commands, the events, and a reference to the Model .
> - In MVVM, the ViewModel is not in charge of updating the data displayed in the UI—thanks to the powerful data-binding engine provided by WPF and Silverlight, the ViewModel doesn’t need to do that .This is because the View is an observer of the ViewModel, so as soon as the ViewModel changes, the UI updates itself .For that to happen, the ViewModel must implement the INotifyPropertyChangedinterface and fire the PropertyChangedevent .

简单翻译一下（不全）
> - The Model 代表业务逻辑的实体类，可以是一个简单的顾客实体类，也可以是一个复杂的股票交易实体类。
> - The View 代表一个用户界面控件 ...
> - The ViewModel 包括各种逻辑、命令、事件以及实体类的引用。


#什么是MVVMFoundation#


[MVVMFoundation](http://mvvmfoundation.codeplex.com)是一个最简单的MVVM框架，官方介绍如下：

>MVVM Foundation is a library of classes that are very useful when building applications based on the Model-View-ViewModel philosophy. The library is small and concentrated on providing only the most indispensable tools needed by most MVVM application developers

MVVMFoundation包含四大模块：

- ObservableObject：这里相当于ViewModelBase的概念，每一个ViewModel继承自该类，调用完成之后立即释放，防止内存泄露。

- RelayCommand接口：封装command的声明，包括execution执行逻辑,可选的can-execute逻辑等。外部只需要实例化并Binding就可以简单使用。

- Messenger:这里主要用在各种不同的ViewModel之间通信（比如相互关联的ViewModel、主从ViewModel等），当然也可以扩展成ViewModel与View之间进行通信。

- PropertyObserver：主要是对INotifyPropertyChanged.PropertyChanged进行封装，可以通过其对某个对象的属性变更注册回调函数，当属性变更时便触发回调函数。


#ObservableObject#
实现ViewModel中的属性改变通知到绑定的控件的方法，相当于是所有Viewmodel的基类。


1. 使用时调用OnPropertyChange方法，则后台数据变化即可通知界面刷新
```           
public string UserName
{
    get { return this.user.UserName; }
    set
        {
            this.user.UserName = value;
	    OnPropertyChanged("UserName");
	}
}
```
2. 属性在View界面的绑定
```           
<TextBox Text="{Binding UserName}" />

```

#RelayCommand#
用于在ViewModel中定义View中绑定的命令，代替了以前Winform的Click事件。

1. 在ViewModel中定义Command
```
public ICommand BrowseImageCommand
{
    get { return new ICommand(BrowseImage); }
}

private void BrowseImage()
{
     ...
}
```

2. 在View中的按钮关联此Command
```
<Button Content="浏览..." Command="{Binding BrowseImageCommand}"/>
```

#Messenger#
可用于ViewModel之间的信息传递，可以用于ViewModel和View之间的信息传递。

1. 定义信息传输类
```
public class ViewModelCommunication
{
    static ViewModelCommunication()
    {
        Messaging = new Messenger();
    }
    public static Messenger Messaging { get; set; }
    public static string DataIDInChanged { get { return "DataIDInChanged"; } }
}
```

2. 在需要通知的类中注册要通知的信息
```
ViewModelCommunication.Messaging.Register(ViewModelCommunication.DataIDInChanged,
        (Action<string>)(param => SetLastSelectedDataID(param))); 
```

3. 当对应的消息出现时，通知已经注册的类
```
ViewModelCommunication.Messaging.NotifyColleagues(ViewModelCommunication.DataIDInChanged, 
        DataID.ToString()); 
```

#PropertyObserver#
主要用于对对象的属性监听，属性变更后可触发已注册的回调函数。

1. 注册要监听对象的属性及回调函数
```
PropertyObserver<UserInfoViewModel> userInfoAfterObserver;
public MainWindowViewModel()
{
    UserInfoBefore = new UserInfoViewModel();
    userInfoAfterObserver = new PropertyObserver<UserInfoViewModel>(UserInfoAfter)
        .RegisterHandler(UserInfo => UserInfo.Age, this.AgeChangedCallback);
}
```

2. 实现回调函数
```
private void AgeChangedCallback(UserInfoViewModel userInfo)
{
    MessageBox.Show("Property Age changed"); 
}
```

以上就是MVVMFoundation框架的主要使用方法，感兴趣的人可以用用看～欢迎留言交流心得～
