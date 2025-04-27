

## WPF资源样式

### 一、WPF如何为项目设置全局样式

在项目中，需要为所有的Button、TextBox设置一个默认的全局样式，一个个的为多个控件设置相同的样式显然是不明智的，在WPF设置全局样式主要有两种写法：

1. 第一种就是先写好按钮的样式，不写Key，然后再App.xaml中引用

```xaml
<ResourceDictionary xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation" xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
    <Style  TargetType="{x:Type CheckBox}" />
</ResourceDictionary>
<!--这里先写一个资源字典，然后在App.xaml中引入即可---!>
```

那么在App.xaml中就应该这样写了

```xaml
<Application x:Class="_2.CheckBox控件.App"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:local="clr-namespace:_2.CheckBox控件"
             StartupUri="MainWindow.xaml">
    <Application.Resources>
        <!--这种是资源字典合并-->
        <ResourceDictionary>
            <ResourceDictionary.MergedDictionaries>
                <!--按钮基本样式--->
                <ResourceDictionary Source="pack://application:,,,/Resources/ButtonStyle/BasicButton.xaml"></ResourceDictionary>
                <!-- CheckBox基本样式--->
                <ResourceDictionary Source="pack://application:,,,/Resources/ButtonStyle/BasicCheckbox.xaml"></ResourceDictionary>
            </ResourceDictionary.MergedDictionaries>
        </ResourceDictionary>
    </Application.Resources>
</Application>


```

这种方式有多少个控件就需要在APP中累砌多少个引用，会使配置文件杂乱冗余，而且由于默认样式没有Key，控制不够灵活，所以再介绍下第二种方法。

2. 为控件写的样式和上文差不多，只是加上Key。(没有Key为全局样式，有Key则需要进行键值引用)

首先我在一个资源字典里面写了两个Button的样式，分别加了Key，也就是唯一标识。接下来我再新建一个资源，统一管理所有的样式资源，通过BaseOn继承带Key的样式，转换为默认全局样式，然后只需要在App中引入一个资源文件即可，这样即使需要写几十个、上百个样式，APP中也只需要一行代码

```xaml
<ResourceDictionary xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
                    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
    
    <Style TargetType="Button"  x:Key="DefaultButton" >
        <Setter Property="HorizontalAlignment" Value="Center"></Setter>
        <Setter Property="VerticalAlignment" Value="Center"></Setter>
        <Setter Property="Foreground" Value="White"></Setter>
        <Setter Property="Background" Value="#1e6fff"></Setter>
        <Setter Property="Width" Value="100"></Setter>
        <Setter Property="Height" Value="30"></Setter>
        <Setter Property="Template">
            <Setter.Value>
                <ControlTemplate TargetType="Button">
                    <Border x:Name="btn" Background="{TemplateBinding Background}" CornerRadius="5">
                        <ContentPresenter HorizontalAlignment="{TemplateBinding HorizontalAlignment}"
                                              VerticalAlignment="{TemplateBinding VerticalAlignment}"></ContentPresenter>
                    </Border>
                    <ControlTemplate.Triggers>
                        <Trigger Property="IsPressed" Value="True">
                            <Setter Property="Background" Value="#0082fc" TargetName="btn"></Setter>
                        </Trigger>
                        <Trigger Property="IsMouseOver" Value="True">
                            <Setter Property="Background" Value="#0087e1" TargetName="btn"></Setter>
                        </Trigger>
                    </ControlTemplate.Triggers>
                </ControlTemplate>
            </Setter.Value>
        </Setter>
    </Style>
    
    <Style TargetType="Button"  x:Key="ErrorButton" >
        <Setter Property="HorizontalAlignment" Value="Center"></Setter>
        <Setter Property="VerticalAlignment" Value="Center"></Setter>
        <Setter Property="Foreground" Value="White"></Setter>
        <Setter Property="Background" Value="#ff0000"></Setter>
        <Setter Property="Width" Value="100"></Setter>
        <Setter Property="Height" Value="30"></Setter>
        <Setter Property="Template">
            <Setter.Value>
                <ControlTemplate TargetType="Button">
                    <Border x:Name="btn" Background="{TemplateBinding Background}" CornerRadius="5">
                        <ContentPresenter HorizontalAlignment="{TemplateBinding HorizontalAlignment}"
                                              VerticalAlignment="{TemplateBinding VerticalAlignment}"></ContentPresenter>
                    </Border>
                    <ControlTemplate.Triggers>
                        <Trigger Property="IsPressed" Value="True">
                            <Setter Property="Background" Value="#e33b2f" TargetName="btn"></Setter>
                        </Trigger>
                        <Trigger Property="IsMouseOver" Value="True">
                            <Setter Property="Background" Value="#db4146" TargetName="btn"></Setter>
                        </Trigger>
                    </ControlTemplate.Triggers>
                </ControlTemplate>
            </Setter.Value>
        </Setter>
    </Style>
</ResourceDictionary>
```

新建一个资源引入这个Button样式的资源

```xaml
<ResourceDictionary xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
                    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
    <ResourceDictionary.MergedDictionaries>
        <ResourceDictionary Source="/Dict/Button.xaml"></ResourceDictionary>
        <ResourceDictionary Source="/Dict/CheckBox.xaml"></ResourceDictionary>
    </ResourceDictionary.MergedDictionaries>
    <Style TargetType="Button" BasedOn="{StaticResource DefaultButton}"></Style>
</ResourceDictionary>
```

让Button基于BaseOn关键字继承DefaultButton这个样式。这样在整个项目中，Button就会使用这个DefaultButton这个样式，如果有其他需求，可以在Button的Style样式里面，写Style = "{StaticSource ErrorButton}"，这样样式就会在更改

接下来就是在App.xaml中引入统一样式资源就可以了

```xaml
<Application x:Class="_1.Button控件.App"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:local="clr-namespace:_1.Button控件"
             StartupUri="MainWindow.xaml">
    <Application.Resources>
        <ResourceDictionary Source="/Dict/Generic.xaml"></ResourceDictionary>
    </Application.Resources>
</Application>

```

### 二、在WPF中资源样式的引入

1. 图片的引入，像是.png、.jpg格式的图片，需要设置为资源

![image-20250423093148157](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20250423093148157.png)

2. SVG格式的文件引入

   SVG是一种基于XML的矢量图，WPF本身并不直接支持SVG格式，在WPF中需要经过转换成XAML才能使用，可以自己手动转换，也可以借助第三方库SharpVectors库

   + **第一步：安装SharpVectors库，在NuGet包管理器安装SharpVectors库**

   ```xaml
   Install-Package SharpVectors
   ```

+ **第二步：在XAML中引入命名空间**

```xaml
xmlns:svg="http://SharpVectors.Renderers.Wpf"
```

+ **第三步：使用SvgImage控件动态加载SVG**

```xaml
使用SvgImage控件动态加载SVG
<svg:SvgImage 
    Source="/Assets/your-image.svg" 
    Width="200" 
    Height="200"
    OverrideColor="Blue" <!-- 可选：动态修改颜色 -->
/>
```

+ **第四步：代码中动态加载SVG（示例）**

```c#
using SharpVectors.Converters;

var svgImage = new SvgImage();
svgImage.Source = new Uri("pack://application:,,,/YourAssembly;component/Images/icon.svg");
yourGrid.Children.Add(svgImage);

```

3. WPF中引入字体资源

1. 图片资源和字体资源导入

1. 设置图片和字体为资源![img](https://cdn.nlark.com/yuque/0/2024/png/34364737/1730443896652-07a50d37-b11e-4637-af7b-71fa12d7c3ac.png)
2. 导入资源输入pack://application:,,,/程序集完整名称;component/资源路径 （**资源完整访问路径**）
3. .Net版本和.NetFramework这两个框架下引用图片资源不同

```c#
 <!--绑定图片资源
 图片是不是资源，使用完整路径-->
 <Image
     Height="30" 
     Width="30" 
     Source="pack://application:,,,/Zhaoxi.ResLession;Component/Assets/Logo.png"></Image>

 <!--字体图标使用资源-->
 <!--FontFamily="pack://application:,,,/Zhaoxi.WPFLesson;component/Assets/#iconfont"-->
 <TextBlock Text="哈哈哈" Margin="30" FontFamily="pack://application:,,,/Zhaoxi.ResLession;Component/Assets/#iconfont">
 </TextBlock>
 
 <!--音视频、Gif  必须复制到本地加载-->
 <MediaElement Source="./1.mp3" Width="400" Height="300"></MediaElement>
 <Button Width="100" Height="100" Content="内容"></Button>
```

4. 音视频和GIF必须复制到本地加载，不能设置成资源

### 三、窗口资源和样式

Window窗体资源的常见属性

```c#
 WindowStartupLocation="CenterScreen"	  窗口启动位置
 WindowStyle="None"						窗体样式；边框、无边框、3D
 Background="Red"						窗体背景
 WindowState="Maximized"				最大化，最小化，正常
 AllowsTransparency="True"				允许窗体设置为透明色
 Background="Transparent"  
 FontSize="12"	支持继承
```

无边框窗体设置的两种方式

#### 1. WindowStyle设置

设置窗体Window的属性WindowStyle为None，就会得到一个无边框窗体，设置完成之后，窗体是不支持移动的。如果需要移动，给Window注册MouseLeftButtonDown事件，然后在事件中调用<font color="Red;">DragMove</font>方法。使用MouseLeftButtonDown这个事件，可以让整个窗体在鼠标左键按下的时候，可以移动。

```c#
private void Window_MouseLeftButtonDown(object sender, MouseButtonEventArgs e)
{
    this.DragMove();
}
```

#### 2. WindowChrome设置

设置窗体的WindowChrome，使用CaptionHeight属性，这个属性时控制标题区（设置WindowChrome后它不显示），接收鼠标响应的高度。除此之外，使用WindowChrome后，使用CornerRadius属性设置窗体圆角。

```xaml
<!--WindowChrome-->
<WindowChrome.WindowChrome>
    <WindowChrome CaptionHeight="30" CornerRadius="5" GlassFrameThickness="0"></WindowChrome>
</WindowChrome.WindowChrome>
```

如果按钮在CaptionHeight高度内，这时点击按钮是失活的，不起作用。让其生效必须在Button按钮上添加附加属性WindowChrome.IsHitTestVisibleInChrome="True"，添加完成后，就可以使用Button按钮了

#### 3. 样式 Style 设置

WPF里面写样式和前端里面的CSS语言很像，可以使用关键字<font color="red">BaseOn</font>来继承样式，这个继承样式和前端里面的CSS继承也挺像的。写样式很令人头疼，需要多写多练。下面代码是Label的样式，很简单的代码。样式可以写在资源中、也可以写在资源字典中、也可以写在

```xaml
<Style TargetType="Label" x:Key="SuccessLabel" BasedOn="{StaticResource DefaultLabel}">
    <Setter Property="Background" Value="LightGreen"></Setter>
    <Setter Property="Foreground" Value="White"></Setter>
    <Setter Property="Template">
        <Setter.Value>
            <ControlTemplate TargetType="Label">
                <!--内容-->
                <Border x:Name="container" CornerRadius="3" 
                        Padding="5" 
                        BorderThickness="1"
                        BorderBrush="#f5f5f5"
                        Background="{TemplateBinding Background}">
                    <ContentPresenter
                        HorizontalAlignment="{TemplateBinding HorizontalAlignment}" 
                        VerticalAlignment="{TemplateBinding VerticalAlignment}"></ContentPresenter>
                </Border>
                <!--触发器-->
                <ControlTemplate.Triggers>
                    <MultiTrigger>
                        <MultiTrigger.Conditions>
                            <Condition Property="IsMouseOver" Value="True"></Condition>
                        </MultiTrigger.Conditions>
                        <MultiTrigger.Setters>
                            <Setter Property="Background" TargetName="container" Value="#4de77b"></Setter>
                        </MultiTrigger.Setters>
                    </MultiTrigger>
                </ControlTemplate.Triggers>
            </ControlTemplate>
        </Setter.Value>
    </Setter>
</Style>

```



![image-20250425205609876](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20250425205609876.png)

