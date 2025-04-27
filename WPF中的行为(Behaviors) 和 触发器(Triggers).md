## WPF中的行为(Behaviors) 和 触发器(Triggers)

## 一 、行为

什么是行为？控件的页面逻辑大都可以被认为是行为，比方说TextBox在聚焦后自动全选，Button点击之后会弹出一个小窗口，Grid按照某种方式排布子控件。行为本质上基于附加属性实现的。行为本身也继承了DependencyObject。一句话行为就是某些控件共同特征的实现。

在WPF中如何使用行为呢？或者自定义行为？这个时候可以下载微软提供的第三方包

```xaml
Microsoft.Xaml.Behaviors.Wpf
```

安装此包之后，在XAML中引入命名控件

```
xmlns:i="http://schemas.microsoft.com/xaml/behaviors"
```

在代码中就可以使用行为，这个第三方包提供了常见的行为，比如说拖动控件

```xaml
<Border Width="100" Height="100" Background="Red">
    <i:Interaction.Behaviors>
        <i:MouseDragElementBehavior></i:MouseDragElementBehavior>
    </i:Interaction.Behaviors>
</Border>
```

这个MouseDragElementBehavior这个行为就可以让Border能够被拖动，这个包提供的默认行为不同，但是开发者可以定制行为的。

如何去定制行为？写一个类继承Behavior<T>这个类提供了两个方法，第一个方法OnAttached()，第二个方法OnDetaching() ，一个对象AssociatedObject 这个对象就是行为附加的对象

<font color="Red">OnAttached()</font> 这个方法意思是把行为附加给某个控件时需要执行的业务逻辑

<font color="Red">OnDetaching()</font> 这个方法就是分离行为需要执行的业务逻辑

假设有这样一个需求，ListBox中添加的数据大于100条就自动清空，并且每次添加一条数据都自动滚动到最后一条，用行为怎么实现？

```C#
    public class ListBehavior : Behavior<ListBox>
    {
        private const int maxCount = 100;

        protected override void OnAttached()
        {
            if (AssociatedObject.Items is INotifyCollectionChanged notifyCollection)
            {
                notifyCollection.CollectionChanged += NotifyCollection_CollectionChanged;
            }
        }

        private void NotifyCollection_CollectionChanged(object? sender, NotifyCollectionChangedEventArgs e)
        {

            if (e.Action == NotifyCollectionChangedAction.Add)
            {
                var item = AssociatedObject.Items[AssociatedObject.Items.Count - 1];
                AssociatedObject.ScrollIntoView(item);
            }

            if (AssociatedObject.Items.Count >= maxCount)
            {
                AssociatedObject.Items.Clear();
            }
        }

        protected override void OnDetaching()
        {
            if (AssociatedObject.Items is INotifyCollectionChanged notifyCollection)
            {
                notifyCollection.CollectionChanged -= NotifyCollection_CollectionChanged;
            }
        }
    }
```

接着另外一个需求，点击一个按钮，让TextBox的内容清空，怎么做呢？

思路：写一个行为，在行为中声明一个依赖对象，这个依赖对象是TextBox类型，绑定按钮的Click事件，接着让TextBox内容清空

```C#
    public class ClearBehavior : Behavior<Button>
    {
        
        /// <summary>
        /// 系统中所有的控件都是依赖对象
        /// </summary>
        public TextBox TextBox
        {
            get { return (TextBox)GetValue(TextBoxProperty); }
            set { SetValue(TextBoxProperty, value); }
        }

        // Using a DependencyProperty as the backing store for TextBox.  This enables animation, styling, binding, etc...
        public static readonly DependencyProperty TextBoxProperty =
            DependencyProperty.Register("TextBox", typeof(TextBox), typeof(ClearBehavior), new PropertyMetadata(null));

        /// <summary>
        /// 附加完行为之后，要执行的业务逻辑
        /// </summary>
        protected override void OnAttached()
        {
            AssociatedObject.Click += ButtonClick;
        }

        /// <summary>
        /// 附加完行为之后，要移除的业务逻辑
        /// </summary>
        protected override void OnDetaching()
        {
            AssociatedObject.Click -= ButtonClick;
        }

        private void ButtonClick(object sender, RoutedEventArgs e)
        {
            TextBox.Clear();
        }
    }
```

在XAML中如何引用的？

```xaml
<WrapPanel Orientation="Vertical" HorizontalAlignment="Center">
    <TextBox x:Name="text" Width="100"></TextBox>
    <Button Content="清空">
        <i:Interaction.Behaviors>
            <behavior:ClearBehavior TextBox="{Binding ElementName=text}"></behavior:ClearBehavior>
        </i:Interaction.Behaviors>
    </Button>
</WrapPanel>
```

## 二、触发器

触发器也是在Behaviors这个包里面，触发器有点类似与WPF中原生的触发器Triggers，在某些条件时会触发一些变化，但是比原生的Triggers更灵活，做的事情也更多

三种经常使用的触发器 CallMethodAction ChangePropertyAction InvokeCommandAction 这三种

1. CallMethodAction 这个触发器会调用某个源对象TargetObject的方法Method

```xaml
这个例子就是说，当我点击之后，会调用源对象Window的Close方法。意思就是说点击按钮，关闭窗体
<Button Content="关闭" Width="100" Height="30">
    <i:Interaction.Triggers>
        <i:EventTrigger EventName="Click">
            <i:CallMethodAction TargetObject="{Binding RelativeSource={RelativeSource Mode=FindAncestor, AncestorType=Window}}" MethodName="Close"></i:CallMethodAction>
        </i:EventTrigger>
    </i:Interaction.Triggers>
</Button>

也可以这样
<Button Content="关闭" Width="100" Height="30">
    <i:Interaction.Triggers>
        <i:EventTrigger EventName="Click">
            <i:CallMethodAction TargetObject="{Binding Source={x:Static Application.Current}}" MethodName="Shutdown"></i:CallMethodAction>
        </i:EventTrigger>
        <i:DataTrigger Binding=""></i:DataTrigger>
    </i:Interaction.Triggers>
</Button>
```

2. InvokeCommandAction 这个触发器就是把事件转成命令Command来调用

   举例：当窗体的Loaded事件触发时，执行LoadCommand这个命令

```xaml
<i:Interaction.Triggers>
    <i:EventTrigger EventName="Loaded">
        <i:InvokeCommandAction Command="{Binding LoadCommand}"></i:InvokeCommandAction>
    </i:EventTrigger>
</i:Interaction.Triggers>
<Window.Resources>
```

3. ChangePropertyAction 这个触发器就是当属性值发生变化时触发

   举例：这个DataTrigger绑定了Button的IsMouseOver这个属性，当属性为True时，更改目标对象Button的属性BackGround为Red

```xaml
<Button Content="点击" Width="100" Height="30">
    <i:Interaction.Triggers>
        <i:DataTrigger Binding="{Binding RelativeSource={RelativeSource AncestorType=Button}, Path=IsMouseOver}"
                       Value="True">
            <i:ChangePropertyAction  TargetObject="{Binding RelativeSource={RelativeSource AncestorType=Button}}" PropertyName="Background" Value="Red"></i:ChangePropertyAction>
        </i:DataTrigger>
    </i:Interaction.Triggers>
</Button>
```

## 三、总结

行为和触发器都是互动的一种表现形式。