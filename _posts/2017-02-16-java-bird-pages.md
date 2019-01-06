---
layout: post
title: JAVA小游戏开发flappy bird-飞扬的小鸟（二）
date: 2017-02-16 11:31:22
categories: java
tags: project
author: MarsHu
---

* content
{:toc}

# 实现窗口中各个对象的移动 #
在前一篇博客中，我们成功的创建了一个游戏窗口，并且在游戏窗口中添加了小鸟、地面、柱子
三个对象，但是，此时的界面是一个静态的界面，各个对象都是无法移动的。

这篇博客中，让我们一步步去实现三个对象的移动效果。分别实现：地面移动、柱子移动、小鸟移动、以及小鸟动画效果。

**在这个教程中，大量使用了`javaswing`里的内容，如果有初学者不是很懂，其实影响也不大。代码其实是写会的，而不是理解会的。**





# 游戏开发步骤（续） #
> ### 步骤七：实现地面的移动 ###

首先，让我们来看下地面是如何移动的，地面的移动效果图（图18）：

![bird18.png](http://marshucheng1.github.io/assets/bird18.png)

从图18我们可以知道，地面初始状态为静止状态，运行游戏时，我们需要地面向左移动。

当地面向左移动到x坐标为-109时，则将x坐标重新设置为0，形成滚动的效果。

我们可以在`Ground`类中定义`step`方法，具体代码：

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

		/*初始化Ground的属性*/
		public Ground() throws Exception {
			image = ImageIO.read(getClass().getResource("ground.png"));
			width = image.getWidth();
			height = image.getHeight();
			x = 0;
			y = 500;
		}
		
		/*游戏运行时,地面向左移动*/
		public void step() {
			x--;
			/*当移动到x坐标为-109时,重置为0*/
			if(x==-109) {
				x=0;
			}
		}

	}

当`step`方法添加后，是不是就实现了柱子的移动效果呢，实际上，并没有。还需要在`BirdGame`类

中添加`action`方法，在方法中利用死循环，在该循环中可以做下操作来实现地面的运动：

**1）修改地面的位置，调用`Ground`类的`step`方法来实现。**

**2）重写绘制界面，调用`Jpanel`的`repaint`方法来实现。**

**3）休眠1/30秒，即设置屏幕的刷新率为1秒30次，调用`Thread`类的`sleep`方法来实现。**

第一个操作，调用`step`方法，是毫无疑问的，如果不调用，地面怎么可能移动呢。

第二个操作，因为游戏界面只是静态的，如果你想实现移动效果，只能重写绘画下游戏界面。这有点

类似皮影戏，静态的东西本身不具备移动效果，只能不停的绘制新的界面，达到欺骗眼睛的效果。

第三个操作，就是绘制界面的频率了，通常用线程睡眠的方式来实现。

具体代码为：

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

		/*添加action方法,实现界面对象的移动效果*/
		public void action() throws Exception {
			while(true) {
				/*修改地面位置*/
				ground.step();
				/*重写绘制界面*/
				repaint();
				/*设置屏幕的刷新率为1秒30次*/
				Thread.sleep(1000/30);
			}
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

在`BirdGame`类的`main`方法中调用`action()`方法，需要在设置窗口可见后调用即写在

`frame.setVisible(true);`之后。（`因为并没有单独启动线程调用`...）

因为我们的游戏程序，实际上是一个单线程程序，所以如果你不是在游戏显示后再去调用的话。

实际上线程已经死在那个死循环中了，连游戏界面都看不见了。关于这个问题，其实也很简单。

就是创建一个新的线程，后面我们可以来改进。完整代码为：

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

		/*添加action方法,实现界面对象的移动效果*/
		public void action() throws Exception {
			while(true) {
				/*修改地面位置*/
				ground.step();
				/*重写绘制界面*/
				repaint();
				/*设置屏幕的刷新率为1秒30次*/
				Thread.sleep(1000/30);
			}
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
			/*调用action*/
			game.action();
		}
	}

此时，我们再次运行游戏时，就会发现，已经实现了地面的移动效果。

> ### 步骤八：实现柱子的移动 ###

要实现柱子的移动，要计算出将柱子移动回出场位置的坐标，具体可以看下图（图19）：

![bird19.png](http://marshucheng1.github.io/assets/bird19.png)

从图19可以看出，当柱子的x坐标为-width/2时，柱子移出界面。

此外，从上图可以看出，移回出场位置的1号柱子的x坐标为distance*2-width/2。

y坐标的范围不变，依然是132到350之间的随机数。

我们可以在`Coloumn`类中添加`step`方法：

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

		/*为柱子添加移动方法*/
		public void step() {
			x--;
			if(x==-width/2) {
				x=distance*2-width/2;
				y=random.nextInt(218)+132;
			}
		}

	}

同样的，我们在`Birdgame`类中的`action`方法中，调用柱子的移动方法，实现柱子移动。

而且，由于在实现地面移动时，我们已经调用过了`action`方法，所以，一旦在`action`方法中

调用了柱子的移动方法，运行游戏后，就能成功看到柱子的移动效果了。主要代码：

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

		/*添加action方法,实现界面对象的移动效果*/
		public void action() throws Exception {
			while(true) {
				/*修改地面位置*/
				ground.step();
				/*修改柱子位置*/
				column1.step();
				column2.step();
				/*重写绘制界面*/
				repaint();
				/*设置屏幕的刷新率为1秒30次*/
				Thread.sleep(1000/30);
			}
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
			/*调用action*/
			game.action();
		}
	}

> ### 步骤九：实现鸟的上抛运动（初始向上移动一段距离） ###

实现上抛运动。游戏启动后，小鸟默认向上移动一段距离。具体效果见下图（图20）：

![bird20.png](http://marshucheng1.github.io/assets/bird20.png)

我们可以计算经过时间t的位移s的计算公式，这个是物理与数学的内容。如果大家忘记的话，

其实并不影响，你只要知道就是这么计算的即可，我们不可能什么都知道。

经过时间t的位置s的公式：`s=v0*t+g*t*t/2`。

从上图可以看出，经过时间t后，小鸟的纵坐标位置y1的计算公式为：`y1=y-s`。

经过时间t后，小鸟的运行速度v为：`v=v0-g*t`。

我们需要在`Bird`类中，增加部分属性，来实现小鸟的默认上抛移动，代码为：

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
		/*在Bird类中增加属性,用于计算鸟的位置*/
		public double g;//重力加速度
		public double t;//两次位置的间隔时间
		public double v0;//初始上抛速度
		public double speed;//当前的上抛速度
		public double s;//经过时间t以后的位移
	
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

定义完这些属性后，我们需要在构造方法中初始化它们，代码为：

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
		/*在Bird类中增加属性,用于计算鸟的位置,实现鸟的上抛*/
		public double g;//重力加速度
		public double t;//两次位置的间隔时间
		public double v0;//初始上抛速度
		public double speed;//当前的上抛速度
		public double s;//经过时间t以后的位移
	
		/*初始化Bird的属性变量*/
		public Bird() throws Exception {
			image = ImageIO.read(getClass().getResource("0.png"));
			width = image.getWidth();
			height = image.getHeight();
			x = 132;
			y = 280;
			size = 40;
			/*初始化上抛移动增加的属性*/
			g=4;
			v0=20;
			t=0.25;
			speed=v0;
			s=0;
		}

	}

初始化属性之后，我们同样的在`Bird`类中添加移动方法，代码如下：

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
		/*在Bird类中增加属性,用于计算鸟的位置,实现鸟的上抛*/
		public double g;//重力加速度
		public double t;//两次位置的间隔时间
		public double v0;//初始上抛速度
		public double speed;//当前的上抛速度
		public double s;//经过时间t以后的位移
	
		/*初始化Bird的属性变量*/
		public Bird() throws Exception {
			image = ImageIO.read(getClass().getResource("0.png"));
			width = image.getWidth();
			height = image.getHeight();
			x = 132;
			y = 280;
			size = 40;
			/*初始化上抛移动增加的属性*/
			g=4;
			v0=20;
			t=0.25;
			speed=v0;
			s=0;
		}

		/*添加鸟的移动方法*/
		public void step() {
			double v0=speed;
			s=v0*t+g*t*t/2;//计算上抛移动的位移
			y=y-(int)s;//计算鸟的坐标位置
			double v=v0-g*t;//计算下次的速度
			speed=v;
		}

	}

同样的，在`BirdGame`类的`action`方法中执行鸟的移动方法，代码如下：

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

		/*添加action方法,实现界面对象的移动效果*/
		public void action() throws Exception {
			while(true) {
				/*修改地面位置*/
				ground.step();
				/*修改柱子位置*/
				column1.step();
				column2.step();
				/*修改鸟的位置(鸟的上抛)*/
				bird.step();
				/*重写绘制界面*/
				repaint();
				/*设置屏幕的刷新率为1秒30次*/
				Thread.sleep(1000/30);
			}
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
			/*调用action*/
			game.action();
		}
	}

> ### 步骤十：实现鸟的振翅效果（小鸟在初始上抛和下落时，翅膀呈挥动状态） ###

要实现这个振翅效果，我们首先肯定需要不同的动态瞬间，在专业的动画制作中，要实现

一个动画效果，通常是一组动画来回切换来实现的，叫做动画帧......。

使用数组存储要显示的图片，共存储8张图片，鸟每运动一次换一张图片显示，以达到鸟的动画效果。

在`Bird`类中添加鸟的振翅动画属性，代码如下：

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
		/*在Bird类中增加属性,用于计算鸟的位置,实现鸟的上抛*/
		public double g;//重力加速度
		public double t;//两次位置的间隔时间
		public double v0;//初始上抛速度
		public double speed;//当前的上抛速度
		public double s;//经过时间t以后的位移
		/*定义一组图片,是鸟的动画帧*/
		public BufferedImage[] images;
		/*定义动画帧数组元素的下标*/
		public int index;

		/*初始化Bird的属性变量*/
		public Bird() throws Exception {
			image = ImageIO.read(getClass().getResource("0.png"));
			width = image.getWidth();
			height = image.getHeight();
			x = 132;
			y = 280;
			size = 40;
			/*初始化上抛移动增加的属性*/
			g=4;
			v0=20;
			t=0.25;
			speed=v0;
			s=0;
		}

		/*添加鸟的移动方法*/
		public void step() {
			double v0=speed;
			s=v0*t+g*t*t/2;//计算上抛移动的位移
			y=y-(int)s;//计算鸟的坐标位置
			double v=v0-g*t;//计算下次的速度
			speed=v;
		}

	}

在`Bird`类的构造方法中初始化鸟的动画属性，代码如下：

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
		/*在Bird类中增加属性,用于计算鸟的位置,实现鸟的上抛*/
		public double g;//重力加速度
		public double t;//两次位置的间隔时间
		public double v0;//初始上抛速度
		public double speed;//当前的上抛速度
		public double s;//经过时间t以后的位移
		/*定义一组图片,是鸟的动画帧*/
		public BufferedImage[] images;
		/*定义动画帧数组元素的下标*/
		public int index;

		/*初始化Bird的属性变量*/
		public Bird() throws Exception {
			image = ImageIO.read(getClass().getResource("0.png"));
			width = image.getWidth();
			height = image.getHeight();
			x = 132;
			y = 280;
			size = 40;
			/*初始化上抛移动增加的属性*/
			g=4;
			v0=20;
			t=0.25;
			speed=v0;
			s=0;
			/*创建具有8个元素的图像数组*/
			images=new BufferedImage[8];
			/*为每个图像设置具体图片*/
			for(int i=0;i<8;i++) {
				images[i]=ImageIO.read(getClass().getResource(i+".png"));
			}
			/*初始下标为0*/
			index=0;
		}

		/*添加鸟的移动方法*/
		public void step() {
			double v0=speed;
			s=v0*t+g*t*t/2;//计算上抛移动的位移
			y=y-(int)s;//计算鸟的坐标位置
			double v=v0-g*t;//计算下次的速度
			speed=v;
		}

	}

在`Bird`类中添加振翅方法`fly`，代码如下：

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
		/*在Bird类中增加属性,用于计算鸟的位置,实现鸟的上抛*/
		public double g;//重力加速度
		public double t;//两次位置的间隔时间
		public double v0;//初始上抛速度
		public double speed;//当前的上抛速度
		public double s;//经过时间t以后的位移
		/*定义一组图片,是鸟的动画帧*/
		public BufferedImage[] images;
		/*定义动画帧数组元素的下标*/
		public int index;

		/*初始化Bird的属性变量*/
		public Bird() throws Exception {
			image = ImageIO.read(getClass().getResource("0.png"));
			width = image.getWidth();
			height = image.getHeight();
			x = 132;
			y = 280;
			size = 40;
			/*初始化上抛移动增加的属性*/
			g=4;
			v0=20;
			t=0.25;
			speed=v0;
			s=0;
			/*创建具有8个元素的图像数组*/
			images=new BufferedImage[8];
			/*为每个图像设置具体图片*/
			for(int i=0;i<8;i++) {
				images[i]=ImageIO.read(getClass().getResource(i+".png"));
			}
			/*初始下标为0*/
			index=0;
		}

		/*添加鸟的移动方法*/
		public void step() {
			double v0=speed;
			s=v0*t+g*t*t/2;//计算上抛移动的位移
			y=y-(int)s;//计算鸟的坐标位置
			double v=v0-g*t;//计算下次的速度
			speed=v;
		}

		/*添加鸟的振翅方法*/
		public void fly() {
			index++;
			image=images[(index)%8];
		}

	}

从实现代码可以看出，每循环到第8张图片，则将`index`的值设为0，即从第一张图片再次开始显示。

`fly`方法实现了鸟的飞翔。一个数与8进行取余数，可以获取0-7之间的余数，正好是`images`中图片的下标。

在fly方法中，如果想要振翅的频率变慢，可以略微修改代码：

	public void fly() {
		index++;
		image=images[(index/12)%8]; 
	}

**大家在实际测试时，可以用这个方法替代fly方法，效果明显一点，后面代码都已经代替了**

在`BirdGame`类的`action`方法中添加鸟的振翅方法:

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

		/*添加action方法,实现界面对象的移动效果*/
		public void action() throws Exception {
			while(true) {
				/*修改地面位置*/
				ground.step();
				/*修改柱子位置*/
				column1.step();
				column2.step();
				/*修改鸟的位置(鸟的上抛)*/
				bird.step();
				/*实现鸟的振翅*/
				bird.fly();
				/*重写绘制界面*/
				repaint();
				/*设置屏幕的刷新率为1秒30次*/
				Thread.sleep(1000/30);
			}
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
			/*调用action*/
			game.action();
		}
	}

再次运行游戏，我们就能看到小鸟的振翅效果了。

> ### 步骤十一：实现鸟的倾斜动画（这个效果比较蛋疼，我也不懂） ###

这个效果很蛋疼，因为之前的小鸟的上抛和下落，以及振翅效果都是属于比较机械的动态效果。

我们还想让鸟在上抛时，鸟头微微朝上......，下落时，鸟头微微朝下.......。这个就是鸟的

倾斜动画，怎么样是不是很意外，具体效果见下图（图21，这个图只是一个抽象的）：

![bird21.png](http://marshucheng1.github.io/assets/bird21.png)

假设水平移动的位移为固定数8，那么倾角=反正切s/8。（`不懂不懂不懂`）

还是让我们就来看看代码具体怎么实现的吧：

**A.首先，我们需要在`Bird`类中，增加倾角变量`alpha`，代码如下：**

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
		/*在Bird类中增加属性,用于计算鸟的位置,实现鸟的上抛*/
		public double g;//重力加速度
		public double t;//两次位置的间隔时间
		public double v0;//初始上抛速度
		public double speed;//当前的上抛速度
		public double s;//经过时间t以后的位移
		/*定义一组图片,是鸟的动画帧*/
		public BufferedImage[] images;
		/*定义动画帧数组元素的下标*/
		public int index;
		/*定义倾角变量*/
		public double alpha;//是鸟的倾度,弧度单位

		/*初始化Bird的属性变量*/
		public Bird() throws Exception {
			image = ImageIO.read(getClass().getResource("0.png"));
			width = image.getWidth();
			height = image.getHeight();
			x = 132;
			y = 280;
			size = 40;
			/*初始化上抛移动增加的属性*/
			g=4;
			v0=20;
			t=0.25;
			speed=v0;
			s=0;
			/*创建具有8个元素的图像数组*/
			images=new BufferedImage[8];
			/*为每个图像设置具体图片*/
			for(int i=0;i<8;i++) {
				images[i]=ImageIO.read(getClass().getResource(i+".png"));
			}
			/*初始下标为0*/
			index=0;
		}

		/*添加鸟的移动方法*/
		public void step() {
			double v0=speed;
			s=v0*t+g*t*t/2;//计算上抛移动的位移
			y=y-(int)s;//计算鸟的坐标位置
			double v=v0-g*t;//计算下次的速度
			speed=v;
		}

		/*添加鸟的振翅方法*/
		public void fly() {
			index++;
			image=images[(index/12)%8];
		}

	}

**B.在`Bird`类的构造方法中，初始化倾角`alpha`：**

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
		/*在Bird类中增加属性,用于计算鸟的位置,实现鸟的上抛*/
		public double g;//重力加速度
		public double t;//两次位置的间隔时间
		public double v0;//初始上抛速度
		public double speed;//当前的上抛速度
		public double s;//经过时间t以后的位移
		/*定义一组图片,是鸟的动画帧*/
		public BufferedImage[] images;
		/*定义动画帧数组元素的下标*/
		public int index;
		/*定义倾角变量*/
		public double alpha;//是鸟的倾度,弧度单位

		/*初始化Bird的属性变量*/
		public Bird() throws Exception {
			image = ImageIO.read(getClass().getResource("0.png"));
			width = image.getWidth();
			height = image.getHeight();
			x = 132;
			y = 280;
			size = 40;
			/*初始化上抛移动增加的属性*/
			g=4;
			v0=20;
			t=0.25;
			speed=v0;
			s=0;
			/*创建具有8个元素的图像数组*/
			images=new BufferedImage[8];
			/*为每个图像设置具体图片*/
			for(int i=0;i<8;i++) {
				images[i]=ImageIO.read(getClass().getResource(i+".png"));
			}
			/*初始下标为0*/
			index=0;
			/*初始化倾角*/
			alpha=0;
		}

		/*添加鸟的移动方法*/
		public void step() {
			double v0=speed;
			s=v0*t+g*t*t/2;//计算上抛移动的位移
			y=y-(int)s;//计算鸟的坐标位置
			double v=v0-g*t;//计算下次的速度
			speed=v;
		}

		/*添加鸟的振翅方法*/
		public void fly() {
			index++;
			image=images[(index/12)%8];
		}

	}

**C.在`Bird`类的`step`方法中，实时倾角`alpha`：**

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
		/*在Bird类中增加属性,用于计算鸟的位置,实现鸟的上抛*/
		public double g;//重力加速度
		public double t;//两次位置的间隔时间
		public double v0;//初始上抛速度
		public double speed;//当前的上抛速度
		public double s;//经过时间t以后的位移
		/*定义一组图片,是鸟的动画帧*/
		public BufferedImage[] images;
		/*定义动画帧数组元素的下标*/
		public int index;
		/*定义倾角变量*/
		public double alpha;//是鸟的倾度,弧度单位

		/*初始化Bird的属性变量*/
		public Bird() throws Exception {
			image = ImageIO.read(getClass().getResource("0.png"));
			width = image.getWidth();
			height = image.getHeight();
			x = 132;
			y = 280;
			size = 40;
			/*初始化上抛移动增加的属性*/
			g=4;
			v0=20;
			t=0.25;
			speed=v0;
			s=0;
			/*创建具有8个元素的图像数组*/
			images=new BufferedImage[8];
			/*为每个图像设置具体图片*/
			for(int i=0;i<8;i++) {
				images[i]=ImageIO.read(getClass().getResource(i+".png"));
			}
			/*初始下标为0*/
			index=0;
			/*初始化倾角*/
			alpha=0;
		}

		/*添加鸟的移动方法*/
		public void step() {
			double v0=speed;
			s=v0*t+g*t*t/2;//计算上抛移动的位移
			y=y-(int)s;//计算鸟的坐标位置
			double v=v0-g*t;//计算下次的速度
			speed=v;
			/*计算倾角:利用Java API提供的方法*/
			alpha=Math.atan(s/8);
		}

		/*添加鸟的振翅方法*/
		public void fly() {
			index++;
			image=images[(index/12)%8];
		}

	}

**D.在`BirdGame`类中修改`paint()`方法，实现鸟的旋转：**

这里需要注意将原来的绘制鸟的方法替换掉，不要保留原来的绘制鸟的方法。

	package testbird;

	import java.awt.image.BufferedImage;
	import javax.swing.JPanel;
	import javax.imageio.ImageIO;
	import javax.swing.JFrame;
	import java.awt.Graphics;
	import java.awt.Graphics2D;

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
			/*实现鸟的倾角动画*/
			Graphics2D g2=(Graphics2D) g;
			g2.rotate(-bird.alpha, bird.x, bird.y);
			/*绘制鸟*/
			g.drawImage(bird.image, bird.x - bird.width/2, bird.y - bird.height/2, null);
			g2.rotate(bird.alpha, bird.x, bird.y);
		}

		/*添加action方法,实现界面对象的移动效果*/
		public void action() throws Exception {
			while(true) {
				/*修改地面位置*/
				ground.step();
				/*修改柱子位置*/
				column1.step();
				column2.step();
				/*修改鸟的位置(鸟的上抛)*/
				bird.step();
				/*实现鸟的振翅*/
				bird.fly();
				/*重写绘制界面*/
				repaint();
				/*设置屏幕的刷新率为1秒30次*/
				Thread.sleep(1000/30);
			}
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
			/*调用action*/
			game.action();
		}
	}

这个时候，我们再次运行游戏时，就能看到小鸟在上抛时，鸟头是倾斜朝上的。在下落时，鸟头是倾斜朝下的。

> ### 步骤十二：实现鼠标点击事件（点击鼠标时，小鸟可以向上移动） ###

按照上面的步骤，我们实现了游戏界面物体的移动，但是，我们此时有个很大的需求，我们希望

能控制小鸟的移动，总不能让小鸟一下子就下落不见了！！！

因此我们需要实现鼠标点击事件，点击鼠标可以控制小鸟运动。

**A.在`Bird`类中添加重置飞行速度的方法**

鼠标点击后，小鸟应该立刻马上停止下落，向上移动。添加重置飞行速度方法，代码如下：

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
		/*在Bird类中增加属性,用于计算鸟的位置,实现鸟的上抛*/
		public double g;//重力加速度
		public double t;//两次位置的间隔时间
		public double v0;//初始上抛速度
		public double speed;//当前的上抛速度
		public double s;//经过时间t以后的位移
		/*定义一组图片,是鸟的动画帧*/
		public BufferedImage[] images;
		/*定义动画帧数组元素的下标*/
		public int index;
		/*定义倾角变量*/
		public double alpha;//是鸟的倾度,弧度单位

		/*初始化Bird的属性变量*/
		public Bird() throws Exception {
			image = ImageIO.read(getClass().getResource("0.png"));
			width = image.getWidth();
			height = image.getHeight();
			x = 132;
			y = 280;
			size = 40;
			/*初始化上抛移动增加的属性*/
			g=4;
			v0=20;
			t=0.25;
			speed=v0;
			s=0;
			/*创建具有8个元素的图像数组*/
			images=new BufferedImage[8];
			/*为每个图像设置具体图片*/
			for(int i=0;i<8;i++) {
				images[i]=ImageIO.read(getClass().getResource(i+".png"));
			}
			/*初始下标为0*/
			index=0;
			/*初始化倾角*/
			alpha=0;
		}

		/*添加鸟的移动方法*/
		public void step() {
			double v0=speed;
			s=v0*t+g*t*t/2;//计算上抛移动的位移
			y=y-(int)s;//计算鸟的坐标位置
			double v=v0-g*t;//计算下次的速度
			speed=v;
			/*计算倾角:利用Java API提供的方法*/
			alpha=Math.atan(s/8);
		}

		/*添加鸟的振翅方法*/
		public void fly() {
			index++;
			image=images[(index/12)%8];
		}

		/*重置鸟的初始速度,重新向上飞*/
		public void flappy() {
			speed = v0;
		}

	}

**B.在`BirdGame`类的`action()`方法中注册监听事件**

为游戏界面增加鼠标监听事件，点击鼠标时，实现鸟的上抛移动。代码如下：

	package testbird;

	import java.awt.image.BufferedImage;
	import javax.swing.JPanel;
	import javax.imageio.ImageIO;
	import javax.swing.JFrame;
	import java.awt.Graphics;
	import java.awt.Graphics2D;
	import java.awt.event.MouseAdapter;
	import java.awt.event.MouseEvent;
	import java.awt.event.MouseListener;

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
			/*实现鸟的倾角动画*/
			Graphics2D g2=(Graphics2D) g;
			g2.rotate(-bird.alpha, bird.x, bird.y);
			/*绘制鸟*/
			g.drawImage(bird.image, bird.x - bird.width/2, bird.y - bird.height/2, null);
			g2.rotate(bird.alpha, bird.x, bird.y);
		}

		/*添加action方法,实现界面对象的移动效果*/
		public void action() throws Exception {
			while(true) {
				/*修改地面位置*/
				ground.step();
				/*修改柱子位置*/
				column1.step();
				column2.step();
				/*修改鸟的位置(鸟的上抛)*/
				bird.step();
				/*实现鸟的振翅*/
				bird.fly();
				/*绑定鼠标监听事件*/
				MouseListener listener = new MouseAdapter() {
					/*鼠标按下*/
					public void mousePressed(MouseEvent e) {
						bird.flappy();
					}
				};
				/*将listener挂接到当前的面板（BirdGame）上*/
				addMouseListener(listener);
				/*重写绘制界面*/
				repaint();
				/*设置屏幕的刷新率为1秒30次*/
				Thread.sleep(1000/30);
			}
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
			/*调用action*/
			game.action();
		}
	}

此时，大家运行游戏时，点击鼠标时，就能实现鸟的上移效果了。


