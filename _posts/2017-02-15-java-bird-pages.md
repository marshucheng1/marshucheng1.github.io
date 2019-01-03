---
layout: post
title: JAVA小游戏开发flappy bird-飞扬的小鸟（一）
date: 2017-02-15 10:15:22
categories: java
tags: project
author: MarsHu
---

* content
{:toc}

# 游戏介绍 #
《flappy bird》是一款由来自越南的独立游戏开发者Dong Nguyen所开发的作品。
游戏于2013年5月24日上线，并在2014年2月突然暴红。

2014年2月，《Flappy Bird》被开发者本人从苹果及谷歌应用商店撤下。
2014年8月份正式回归APP STORE，正式加入Flappy迷们期待已久的多人对战模式。

游戏中玩家必须控制一只小鸟，跨越由各种不同长度水管所组成的障碍。

游戏初始界面（图1）：

![bird1.png](http://marshucheng1.github.io/assets/bird1.png)





游戏结束界面（图2）：

![bird2.png](http://marshucheng1.github.io/assets/bird2.png)
# 游戏玩法 #
玩家在游戏初始界面（图1），任意位置点击鼠标左键，开始游戏。

游戏开始后，玩家需要不断控制点击屏幕频率来调节小鸟的飞行高度与下降速度。让小鸟通过游戏界面中柱子间的缝隙，每通过一缝隙，会加1分。

如果小鸟不小心碰到柱子，或者掉落地上，游戏宣告结束，见游戏结束界面（图2）。玩家可以在游戏结束界面任意位置按下鼠标左键，重新开始游戏。


# 游戏开发方案 #
> ### 需求 ###

飞扬小鸟的项目需求，当然就是上面说的游戏介绍和游戏玩法了。不多说。
> ### 业务对象分析 ###

本项目中业务对象见下图（图3）：

![bird3.png](http://marshucheng1.github.io/assets/bird3.png)

项目由4个对象组成，游戏主界面对象、小鸟对象、柱子对象以及地面对象。

小鸟对象、柱子对象、地面对象在主界面对象中作用。
> ### 业务对象分析 ###

使用绘图坐标系作为游戏对象参考模型。在计算机中，向下的y轴为正，向右的x轴为正。

鸟是正方形区域、柱子是长方形区域，中间有间隙；地面是矩形。

在坐标系中图片的坐标原点（x,y）是以图片左上角为原点计算，而不是图片中心。

额，在本案例中，柱子、小鸟的（x,y）指的是图片中心点，地面的（x,y）指的左上角点。

所有的图片素材会在最后统一放出！！！
> ### 类的设计（包括类的具体属性和方法） ###

这里的属性和方法是全部的属性和方法，我们只需要按步骤开发即可。

游戏主界面`BirdGame`（图4）：

![bird4.png](http://marshucheng1.github.io/assets/bird4.png)

小鸟`Bird`（图5）：

![bird5.png](http://marshucheng1.github.io/assets/bird5.png)

柱子`Column`（图6）：

![bird6.png](http://marshucheng1.github.io/assets/bird6.png)

地面`Ground`（图7）：

![bird7.png](http://marshucheng1.github.io/assets/bird7.png)

# 游戏开发步骤（绘制游戏界面） #

> ### 步骤一：新建工程和包，工程名`flybird` ###

在项目中，包的名字一般为公司域名倒过来，再加上项目名称，即为包名。

例如本案例中，可以起包名：com.study.bird。当然也可以随便起......

> ### 步骤二：构建工程的结构 ###

本项目包名为`testbird`，首先，新建类`BirdGame`，该类由`public`修饰并且该类继承自`Jpanel`类，扩展`Java AWT`的面板功能；

然后，新建`Ground`类，表示游戏中的地面；新建`Column`表示游戏中的柱子；新建`Bird`表示游戏中的鸟。

游戏中的各个对象如图所示（图8）：

![bird8.png](http://marshucheng1.github.io/assets/bird8.png)

各个对象的代码：

`BirdGame`类：

	package testbird;

	import javax.swing.JPanel;

	/*游戏主界面*/
	public class BirdGame extends JPanel{
		/*启动软件的方法*/
		public static void main(String[] args) {

		}
	}

`Ground`地面类：

	package testbird;

	/*地面*/
	public class Ground {

	}

`Column`地面类：

	package testbird;

	/*柱子类型,x、y是柱子类型中心位置*/
	public class Column {

	}

`Bird`鸟类：

	package testbird;

	/*鸟类型,x、y是鸟类型中心位置*/
	public class Bird {

	}

> ### 步骤三：为各个类添加一些属性 ###

### 1.为`BirdGame`类添加属性，我们需要在游戏主界面添加鸟、柱子、地面以及游戏背景界面。 ###

	package testbird;

	import java.awt.image.BufferedImage;
	import javax.swing.JPanel;

	public class BirdGame extends JPanel{
		/*在界面添加鸟*/
		public Bird bird;
		/*在界面添加两个柱子的左边柱子*/
		public Column column1;
		/*在界面添加两个柱子的右边柱子*/
		public Column column2;
		/*在界面添加地面*/
		public Ground ground;
		/*主界面背景图片*/
		public BufferedImage background;
		/*启动软件的方法*/
		public static void main(String[] args) {
			
		}
	}

### 2.在为`Ground`类添加属性前，我们先来看看`Ground`类的属性图解（图9）： ###

![bird9.png](http://marshucheng1.github.io/assets/bird9.png)

从图9我们可以看到，`Ground`类图片坐标（x,y）以图片的左上角为坐标，需要坐标属性。

`Ground`类图片还应该具有宽度`width`和高度`height`属性。

理所当然的`Ground`类还应该具有背景图片属性，我们用`BufferedImage`来存储图片。

	package testbird;

	import java.awt.image.BufferedImage;

	/*地面*/
	public class Ground {
		/*地面图片*/
		public BufferedImage image;
		/*地面图片x坐标*/
		public int x;
		/*地面图片y坐标*/
		public int y;
		/*地面图片宽度*/
		public int width;
		/*地面图片高度*/
		public int height;
	}

### 3.在为`Column`类添加属性前，我们先来看看`Column`类的属性图解（图10）： ###

![bird10.png](http://marshucheng1.github.io/assets/bird10.png)

从图9我们可以看到，`Column`类图片坐标（x,y）以图片的中心点为坐标，需要坐标属性。

`Column`类图片同样应该具有宽度`width`和高度`height`属性。

`Column`类图片上下柱子间有间距`gap`属性，两个不同柱子间有距离`distance`属性。

理所当然的`Column`类同样应该具有背景图片属性，我们用`BufferedImage`来存储图片。

	package testbird;

	import java.awt.image.BufferedImage;

	/*柱子类型,x、y是柱子的中心点位置*/
	public class Column {
		/*柱子图片*/
		public BufferedImage image;
		/*柱子图片x坐标*/
		public int x;
		/*柱子图片y坐标*/
		public int y;
		/*柱子图片宽度*/
		public int width;
		/*柱子图片高度*/
		public int height;
		/*上下柱子中间的缝隙*/
		public int gap;
		/*两个柱子之间的距离*/
		public int distance;
	}

### 4.为`Bird`类添加属性 ###

`Bird`的属性`image`，表示为`Bird`类的贴图（即背景图片）；（x,y）表示鸟类型的中心位置。

`width`，`height`指鸟图片的宽和高；本案例中，小鸟的大小用一个正方形区域来表示。

`size`即表示该正方形的边长，用于后续的碰撞检测。

	package testbird;

	import java.awt.image.BufferedImage;

	/*鸟类型,x、y是鸟类型中心位置*/
	public class Bird {
		/*鸟图片*/
		public BufferedImage image;
		/*鸟图片x坐标*/
		public int x;
		/*鸟图片y坐标*/
		public int y;
		/*鸟图片宽度*/
		public int width;
		/*鸟图片高度*/
		public int height;
		/*鸟的大小,用于碰撞检测,这里用正方形来表示,size表示边长*/
		public int size;
	}

> ### 步骤四：在主界面绘制鸟、柱子、地面（初始化各类） ###

### 1.初始化`BirdGame`类，在构造方法中初始化各个属性。 ###

这里会出现2行错误，因为我们还没有在`Column`类中添加带参的构造方法。

	package testbird;

	import java.awt.image.BufferedImage;
	import javax.swing.JPanel;
	import javax.imageio.ImageIO;

	public class BirdGame extends JPanel{
		/*在界面添加鸟*/
		public Bird bird;
		/*在界面添加两个柱子的左边柱子*/
		public Column column1;
		/*在界面添加两个柱子的右边柱子*/
		public Column column2;
		/*在界面添加地面*/
		public Ground ground;
		/*主界面背景图片*/
		public BufferedImage background;

		/*初始化BirdGame的属性变量*/
		public BirdGame() throws Exception {
			bird = new Bird();
			column1 = new Column(1);
			column2 = new Column(2);
			ground = new Ground();
			/*初始化游戏背景图片*/
			background = ImageIO.read(getClass().getResource("bg.png"));
		}

		/*启动软件的方法*/
		public static void main(String[] args) {
			
		}
	}

### 2.初始化`Column`类，在构造方法中初始化各个属性。 ###

**A.在初始化`Column`类之前，我们先来看看怎么计算`Column`类的`x`坐标（图11）：**

![bird11.png](http://marshucheng1.github.io/assets/bird11.png)

由图11可以看出，第一个柱子的x坐标`x1=432+118=550`，第二根柱子的x坐标`x2=550+245`。

其中，432为界面宽度，245为两根柱子间的固定距离。

因此，柱子的x坐标可以总结为：`x=550+(n-1)*245`，其中n表示第几根柱子，例如：1，2。

**B.然后我们再来看看怎么计算`Column`类的`y`坐标（图12）：**

![bird12.png](http://marshucheng1.github.io/assets/bird12.png)

由图12可以看出，柱子y坐标范围规定为132到350之间的随机数。

使用`Random`类的`nextInt`方法产生[0,218)之间的随机数，再加上132，

由此计算出来的范围即为132到350之间。代码为：y=random.nextInt(218)+132。

这里图片中有一块柱子下部分透明了，希望大家不要误会，实际上，无论柱子的y坐标怎么变，柱子

的上下两个部分在游戏中并不会出现这种透明现象，因为柱子实际上是非常长的！！！

代码为：

	package testbird;

	import java.awt.image.BufferedImage;
	import java.util.Random;
	import javax.imageio.ImageIO;

	/*柱子类型,x、y是柱子的中心点位置*/
	public class Column {
		/*柱子图片*/
		public BufferedImage image;
		/*柱子图片x坐标*/
		public int x;
		/*柱子图片y坐标*/
		public int y;
		/*柱子图片宽度*/
		public int width;
		/*柱子图片高度*/
		public int height;
		/*上下柱子中间的缝隙*/
		public int gap;
		/*两个柱子之间的距离*/
		public int distance;

		/*构造随机函数*/
		public Random random = new Random();

		/*构造器:初始化数据,n代表第几个柱子*/
		public Column(int n) throws Exception {
			image = ImageIO.read(getClass().getResource("column.png"));
			width = image.getWidth();
			height = image.getHeight();
			gap = 144;
			distance = 245;
			x = 550+(n-1)*distance;
			y= random.nextInt(218)+132;
		}

	}

这个时候，我们也会发现`BirdGame`类中已经不报错了，因为`Column`类添加了有参构造函数。

### 3.初始化`Ground`类，在构造方法中初始化各个属性。 ###

在初始化`Ground`类属性前，让我们一起来看看`Ground`类的属性图解（图13）：

![bird13.png](http://marshucheng1.github.io/assets/bird13.png)

通过图13我们可以发现，`Ground`类的属性相对来说还是很简单的，代码为：

	package testbird;

	import java.awt.image.BufferedImage;
	import javax.imageio.ImageIO;

	/*地面*/
	public class Ground {
		/*地面图片*/
		public BufferedImage image;
		/*地面图片x坐标*/
		public int x;
		/*地面图片y坐标*/
		public int y;
		/*地面图片宽度*/
		public int width;
		/*地面图片高度*/
		public int height;

		/**初始化Ground的属性*/
		public Ground() throws Exception {
			image = ImageIO.read(getClass().getResource("ground.png"));
			width = image.getWidth();
			height = image.getHeight();
			x = 0;
			y = 500;
		}

	}

### 4.初始化`Bird`类，在构造方法中初始化各个属性。 ###

在初始化`Bird`类属性前，让我们一起来看看`Bird`类的属性图解（图14）：

![bird14.png](http://marshucheng1.github.io/assets/bird14.png)

通过图14我们可以发现，`Bird`类的属性相对来说还是很简单的，代码为：

	package testbird;

	import java.awt.image.BufferedImage;
	import javax.imageio.ImageIO;

	/*鸟类型,x、y是鸟类型中心位置*/
	public class Bird {
		/*鸟图片*/
		public BufferedImage image;
		/*鸟图片x坐标*/
		public int x;
		/*鸟图片y坐标*/
		public int y;
		/*鸟图片宽度*/
		public int width;
		/*鸟图片高度*/
		public int height;
		/*鸟的大小,用于碰撞检测,这里用正方形来表示,size表示边长*/
		public int size;
		
		/*初始化Bird的属性变量*/
		public Bird() throws Exception {
			image = ImageIO.read(getClass().getResource("0.png"));
			width = image.getWidth();
			height = image.getHeight();
			x = 132;
			y = 280;
			size = 40;
		}

	}

> ### 步骤五：设置主界面 ###

在步骤四中，我们在初始化`BirdGame`类时，并没有设置相关界面属性。编写`BirdGame`类的`main`方法，

在`main`方法中设置窗口的大小、居中、点击窗口右上角'x号'来关闭窗口以及设置窗口可见，代码如下：

	package testbird;

	import java.awt.image.BufferedImage;
	import javax.swing.JPanel;
	import javax.imageio.ImageIO;
	import javax.swing.JFrame;

	public class BirdGame extends JPanel{
		/*在界面添加鸟*/
		public Bird bird;
		/*在界面添加两个柱子的左边柱子*/
		public Column column1;
		/*在界面添加两个柱子的右边柱子*/
		public Column column2;
		/*在界面添加地面*/
		public Ground ground;
		/*主界面背景图片*/
		public BufferedImage background;

		/*初始化BirdGame的属性变量*/
		public BirdGame() throws Exception {
			bird = new Bird();
			column1 = new Column(1);
			column2 = new Column(2);
			ground = new Ground();
			/*初始化游戏背景图片*/
			background = ImageIO.read(getClass().getResource("bg.png"));
		}

		/*启动软件的方法*/
		public static void main(String[] args) throws Exception {
			/*初始化一个操作窗口*/
			JFrame frame = new JFrame();
			/*初始化游戏主面板*/
			BirdGame game = new BirdGame();
			/*将游戏主界面面板添加到操作窗口中*/
			frame.add(game);
			/*设置主界面宽、高*/
			frame.setSize(432, 644);
			/*让主界面在电脑屏幕居中显示*/
			frame.setLocationRelativeTo(null);
			/*设置窗口关闭方式,点窗口关闭x号,java程序也自动结束运行*/
			frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
			/*设置窗口显示可见*/
			frame.setVisible(true);
		}
	}

> ### 步骤六：绘制界面 ###

绘制界面，我们需要重写paint方法（没有为什么，固定绘图方法）。

### 1.绘制界面背景 ###

	package testbird;

	import java.awt.image.BufferedImage;
	import javax.swing.JPanel;
	import javax.imageio.ImageIO;
	import javax.swing.JFrame;
	import java.awt.Graphics;

	public class BirdGame extends JPanel{
		/*在界面添加鸟*/
		public Bird bird;
		/*在界面添加两个柱子的左边柱子*/
		public Column column1;
		/*在界面添加两个柱子的右边柱子*/
		public Column column2;
		/*在界面添加地面*/
		public Ground ground;
		/*主界面背景图片*/
		public BufferedImage background;

		/*初始化BirdGame的属性变量*/
		public BirdGame() throws Exception {
			bird = new Bird();
			column1 = new Column(1);
			column2 = new Column(2);
			ground = new Ground();
			/*初始化游戏背景图片*/
			background = ImageIO.read(getClass().getResource("bg.png"));
		}

		/*重写paint方法,绘制界面*/
		public void paint(Graphics g) {
			/*绘制背景*/
			g.drawImage(background, 0, 0, null);
		}

		/*启动软件的方法*/
		public static void main(String[] args) throws Exception {
			/*初始化一个操作窗口*/
			JFrame frame = new JFrame();
			/*初始化游戏主面板*/
			BirdGame game = new BirdGame();
			/*将游戏主界面面板添加到操作窗口中*/
			frame.add(game);
			/*设置主界面宽、高*/
			frame.setSize(432, 644);
			/*让主界面在电脑屏幕居中显示*/
			frame.setLocationRelativeTo(null);
			/*设置窗口关闭方式,点窗口关闭x号,java程序也自动结束运行*/
			frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
			/*设置窗口显示可见*/
			frame.setVisible(true);
		}
	}

### 2.绘制柱子 ###

绘制柱子时，因为我们定义的柱子的坐标（x,y）并不是规定的图片原点（图片左上角）。所以我们需要进行计算。

x坐标计算图解（图15）：

![bird15.png](http://marshucheng1.github.io/assets/bird15.png)

通过图15，我们可以知道绘制x坐标起始位置为：`x=柱子.x-1/2柱子.width`;

即柱子中心点的x坐标减去柱子图片宽度的一半。

y坐标相对来说要抽象一点，y坐标计算图解（图16）：

![bird16.png](http://marshucheng1.github.io/assets/bird16.png)

通过图16，我们可以知道绘制y坐标起始位置为：`y=柱子.y-1/2柱子.height`;

即柱子中心点的y坐标减去柱子图片高度的一半。

代码为：

	package testbird;

	import java.awt.image.BufferedImage;
	import javax.swing.JPanel;
	import javax.imageio.ImageIO;
	import javax.swing.JFrame;
	import java.awt.Graphics;

	public class BirdGame extends JPanel{
		/*在界面添加鸟*/
		public Bird bird;
		/*在界面添加两个柱子的左边柱子*/
		public Column column1;
		/*在界面添加两个柱子的右边柱子*/
		public Column column2;
		/*在界面添加地面*/
		public Ground ground;
		/*主界面背景图片*/
		public BufferedImage background;

		/*初始化BirdGame的属性变量*/
		public BirdGame() throws Exception {
			bird = new Bird();
			column1 = new Column(1);
			column2 = new Column(2);
			ground = new Ground();
			/*初始化游戏背景图片*/
			background = ImageIO.read(getClass().getResource("bg.png"));
		}

		/*重写paint方法,绘制界面*/
		public void paint(Graphics g) {
			/*绘制背景*/
			g.drawImage(background, 0, 0, null);
			/*绘制柱子*/
			g.drawImage(column1.image, column1.x-column1.width/2, column1.y-column1.height/2, null);
			g.drawImage(column2.image, column2.x-column2.width/2, column2.y-column2.height/2, null);

		}

		/*启动软件的方法*/
		public static void main(String[] args) throws Exception {
			/*初始化一个操作窗口*/
			JFrame frame = new JFrame();
			/*初始化游戏主面板*/
			BirdGame game = new BirdGame();
			/*将游戏主界面面板添加到操作窗口中*/
			frame.add(game);
			/*设置主界面宽、高*/
			frame.setSize(432, 644);
			/*让主界面在电脑屏幕居中显示*/
			frame.setLocationRelativeTo(null);
			/*设置窗口关闭方式,点窗口关闭x号,java程序也自动结束运行*/
			frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
			/*设置窗口显示可见*/
			frame.setVisible(true);
		}
	}

### 3.绘制地面和鸟 ###

这里在绘制鸟的时候，因为鸟定义的坐标（x,y）同样是图片中心点，并不是规定的图片原点（图片左上角）。所以我们需要进行计算。

当然鸟的坐标原点的计算相对来说比较简单。因为是固定正方形。

	package testbird;

	import java.awt.image.BufferedImage;
	import javax.swing.JPanel;
	import javax.imageio.ImageIO;
	import javax.swing.JFrame;
	import java.awt.Graphics;

	public class BirdGame extends JPanel{
		/*在界面添加鸟*/
		public Bird bird;
		/*在界面添加两个柱子的左边柱子*/
		public Column column1;
		/*在界面添加两个柱子的右边柱子*/
		public Column column2;
		/*在界面添加地面*/
		public Ground ground;
		/*主界面背景图片*/
		public BufferedImage background;

		/*初始化BirdGame的属性变量*/
		public BirdGame() throws Exception {
			bird = new Bird();
			column1 = new Column(1);
			column2 = new Column(2);
			ground = new Ground();
			/*初始化游戏背景图片*/
			background = ImageIO.read(getClass().getResource("bg.png"));
		}

		/*重写paint方法,绘制界面*/
		public void paint(Graphics g) {
			/*绘制背景*/
			g.drawImage(background, 0, 0, null);
			/*绘制柱子*/
			g.drawImage(column1.image, column1.x-column1.width/2, column1.y-column1.height/2, null);
			g.drawImage(column2.image, column2.x-column2.width/2, column2.y-column2.height/2, null);
			/*绘制地面*/
			g.drawImage(ground.image, ground.x, ground.y, null);
			/*绘制鸟*/
			g.drawImage(bird.image, bird.x-bird.width/2, bird.y-bird.height/2, null);

		}

		/*启动软件的方法*/
		public static void main(String[] args) throws Exception {
			/*初始化一个操作窗口*/
			JFrame frame = new JFrame();
			/*初始化游戏主面板*/
			BirdGame game = new BirdGame();
			/*将游戏主界面面板添加到操作窗口中*/
			frame.add(game);
			/*设置主界面宽、高*/
			frame.setSize(432, 644);
			/*让主界面在电脑屏幕居中显示*/
			frame.setLocationRelativeTo(null);
			/*设置窗口关闭方式,点窗口关闭x号,java程序也自动结束运行*/
			frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
			/*设置窗口显示可见*/
			frame.setVisible(true);
		}
	}

> ### 运行程序显示界面效果 ###

在运行之前需要更改BirdGame类中的frame.setSize(432, 644);改为frame.setSize(864, 644);

因为，柱子默认是绘制在主界面之外的，所以需要扩大界面宽度，才能完整显示。

这里大家显示的可能略有不同，因为每个柱子y坐标是随机的，界面效果如图（图17）：

![bird17.png](http://marshucheng1.github.io/assets/bird17.png)

最后先把项目中所有用到的图片放出（突然觉得这样放出图片素材是最简单的......）：

![0.png](http://marshucheng1.github.io/assets/bird/0.png)
![1.png](http://marshucheng1.github.io/assets/bird/1.png)
![2.png](http://marshucheng1.github.io/assets/bird/2.png)
![3.png](http://marshucheng1.github.io/assets/bird/3.png)
![4.png](http://marshucheng1.github.io/assets/bird/4.png)
![5.png](http://marshucheng1.github.io/assets/bird/5.png)
![6.png](http://marshucheng1.github.io/assets/bird/6.png)
![7.png](http://marshucheng1.github.io/assets/bird/7.png)
![bg.png](http://marshucheng1.github.io/assets/bird/bg.png)
![column.png](http://marshucheng1.github.io/assets/bird/column.png)
![gameover.png](http://marshucheng1.github.io/assets/bird/gameover.png)
![ground.png](http://marshucheng1.github.io/assets/bird/ground.png)
![start.png](http://marshucheng1.github.io/assets/bird/start.png)