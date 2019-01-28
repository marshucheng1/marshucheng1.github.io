---
layout: post
title: JAVA小游戏开发flappy bird-飞扬的小鸟（三）
date: 2017-02-17 17:31:22
categories: java
tags: project
author: MarsHu
---

* content
{:toc}

# 实现游戏计分、碰撞检测、游戏启动控制 #
在前一篇博客中，我们成功的实现了鼠标控制小鸟的运动效果，并且实现了小鸟的初始上抛和初始下落的效果。

这篇博客中，让我们一步步去实现游戏的最后几个功能。分别实现：计分、碰撞、游戏循环控制等。

**在这个教程中，大量使用了`javaswing`里的内容，如果有初学者不是很懂，其实影响也不大。代码其实是写会的，而不是理解会的。**





# 游戏开发步骤（续） #
> ### 步骤十三：实现界面计分功能 ###

让我们先来看下游戏的计分逻辑是什么：

在游戏中我们人为规定。当`柱子的x坐标`与`鸟的x坐标`重合时，则加一分（这里先不考虑碰撞）。

我们可以通过以下几个步骤实现计分功能：

**1）在`BirdGame`类中增加属性`score`，用于计分，`score`初始化为`0`。**

**2）在`BirdGame`类的`action`方法的主循环中，添加判分逻辑。**

**3）在`BirdGame`类的`paint`方法中将分数画出来。**

第一步的代码为：

	package testbird;

	import java.awt.Graphics;
	import java.awt.Graphics2D;
	import java.awt.event.MouseAdapter;
	import java.awt.event.MouseEvent;
	import java.awt.event.MouseListener;
	import java.awt.image.BufferedImage;
	import javax.swing.JFrame;
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
		/*分数*/
		public int score;

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
				/*重新绘制界面*/
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
			game.action();
		}
	}


第二步的代码为（这里计分逻辑应该写在`repaint()`方法前）：

	package testbird;

	import java.awt.Graphics;
	import java.awt.Graphics2D;
	import java.awt.event.MouseAdapter;
	import java.awt.event.MouseEvent;
	import java.awt.event.MouseListener;
	import java.awt.image.BufferedImage;
	import javax.swing.JFrame;
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
		/*分数*/
		public int score;

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
				/*计分逻辑*/				
				if(bird.x==column1.x || bird.x==column2.x) {
					score++;
				}
				/*重新绘制界面*/
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
			game.action();
		}
	}

第三步的代码为：

	package testbird;

	import java.awt.Color;
	import java.awt.Font;
	import java.awt.Graphics;
	import java.awt.Graphics2D;
	import java.awt.event.MouseAdapter;
	import java.awt.event.MouseEvent;
	import java.awt.event.MouseListener;
	import java.awt.image.BufferedImage;
	import javax.swing.JFrame;
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
		/*分数*/
		public int score;

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
			/*绘制分数*/
			Font f=new Font(Font.SANS_SERIF, Font.BOLD, 40);
			g.setFont(f);
			g.drawString(""+score, 40, 60);
			g.setColor(Color.WHITE);
			g.drawString(""+score, 40-3, 60-3);

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
				/*计分逻辑*/				
				if(bird.x==column1.x || bird.x==column2.x) {
					score++;
				}
				/*重新绘制界面*/
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
			game.action();
		}
	}

> ### 步骤十四：实现鸟的碰撞检测：与地面的碰撞检测 ###

让我们先来看下开发思路：

**1）为`BirdGame`类增加`boolean`变量`gameOver`，用于标识游戏是否结束，`true`表示游戏结束，`false`为游戏还没开始。默认值为`false`。**

**2）当检测到碰到地面时，即`gameOver`的值为`true`。在主循环中检测是否碰到地面。在`Bird`类中增加方法`hit(Ground)`检测鸟是否碰地面。当鸟的y坐标加上size/2大于等于地面的y坐标时，则与地面发生碰撞。**

**3）修改`BirdGame`类的`action`方法中的主循环，当游戏没有结束时，执行鸟和柱子的移动。也就是说，如果游戏结束了，鸟和柱子就不移动了。**

**4）在`BirdGame`类的`paint`方法添加当`gameOver`的值为`true`时，显示游戏结束界面。**

思路有了，然后让我们一起按照步骤开发吧。

①首先，在`BirdGame`类中增加游戏结束开关属性，游戏结束画面属性：

	package testbird;

	import java.awt.Color;
	import java.awt.Font;
	import java.awt.Graphics;
	import java.awt.Graphics2D;
	import java.awt.event.MouseAdapter;
	import java.awt.event.MouseEvent;
	import java.awt.event.MouseListener;
	import java.awt.image.BufferedImage;
	import javax.swing.JFrame;
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
		/*分数*/
		public int score;
		/*游戏结束*/
		public boolean gameOver;//游戏是否结束
		public BufferedImage gameOverImage;//游戏结束画面

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
			/*绘制分数*/
			Font f=new Font(Font.SANS_SERIF, Font.BOLD, 40);
			g.setFont(f);
			g.drawString(""+score, 40, 60);
			g.setColor(Color.WHITE);
			g.drawString(""+score, 40-3, 60-3);

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
				/*计分逻辑*/				
				if(bird.x==column1.x || bird.x==column2.x) {
					score++;
				}
				/*重新绘制界面*/
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
			game.action();
		}
	}

②定义了游戏结束属性后，在`BirdGame`类的构造方法中，初始化游戏结束属性：

	package testbird;

	import java.awt.Color;
	import java.awt.Font;
	import java.awt.Graphics;
	import java.awt.Graphics2D;
	import java.awt.event.MouseAdapter;
	import java.awt.event.MouseEvent;
	import java.awt.event.MouseListener;
	import java.awt.image.BufferedImage;
	import javax.swing.JFrame;
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
		/*分数*/
		public int score;
		/*游戏结束*/
		public boolean gameOver;//游戏是否结束
		public BufferedImage gameOverImage;//游戏结束画面

		/*初始化BirdGame的属性变量*/
		public BirdGame() throws Exception {
			bird = new Bird();
			column1 = new Column(1);
			column2 = new Column(2);
			ground = new Ground();
			/*初始化游戏背景图片*/
			background = ImageIO.read(getClass().getResource("bg.png"));
			/*初始化游戏结束*/
			gameOver=false;
			gameOverImage= ImageIO.read(getClass().getResource("gameover.png"));
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
			/*绘制分数*/
			Font f=new Font(Font.SANS_SERIF, Font.BOLD, 40);
			g.setFont(f);
			g.drawString(""+score, 40, 60);
			g.setColor(Color.WHITE);
			g.drawString(""+score, 40-3, 60-3);

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
				/*计分逻辑*/				
				if(bird.x==column1.x || bird.x==column2.x) {
					score++;
				}
				/*重新绘制界面*/
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
			game.action();
		}
	}

③在`Bird`类中添加碰撞地面检测方法（这里有些代码不好理解，不用理会）：

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
	
		/*碰撞地面检测方法*/
		public boolean hit(Ground ground) {
			boolean hit = y+size/2>ground.y;
			if(hit) {
				/*将鸟放置到地面上*/
				y=ground.y-size/2;
				/*使鸟摔倒在地面上有摔倒效果*/
				/*只是让鸟有一个动画效果,不清楚的可以注释掉仔细观察*/
				alpha= -3.14159265358979323/2;
			}
			return hit;
		}

	}


④重写`BirdGame`类中`action`方法中`while`循环中的部分代码：

	package testbird;

	import java.awt.Color;
	import java.awt.Font;
	import java.awt.Graphics;
	import java.awt.Graphics2D;
	import java.awt.event.MouseAdapter;
	import java.awt.event.MouseEvent;
	import java.awt.event.MouseListener;
	import java.awt.image.BufferedImage;
	import javax.swing.JFrame;
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
		/*分数*/
		public int score;
		/*游戏结束*/
		public boolean gameOver;//游戏是否结束
		public BufferedImage gameOverImage;//游戏结束画面

		/*初始化BirdGame的属性变量*/
		public BirdGame() throws Exception {
			bird = new Bird();
			column1 = new Column(1);
			column2 = new Column(2);
			ground = new Ground();
			/*初始化游戏背景图片*/
			background = ImageIO.read(getClass().getResource("bg.png"));
			/*初始化游戏结束*/
			gameOver=false;
			gameOverImage= ImageIO.read(getClass().getResource("gameover.png"));
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
			/*绘制分数*/
			Font f=new Font(Font.SANS_SERIF, Font.BOLD, 40);
			g.setFont(f);
			g.drawString(""+score, 40, 60);
			g.setColor(Color.WHITE);
			g.drawString(""+score, 40-3, 60-3);

		}
	
		/*添加action方法,实现界面对象的移动效果*/
		public void action() throws Exception {
			while(true) {
				if(!gameOver) {
					/*修改地面位置*/
					ground.step();
					/*修改柱子位置*/
					column1.step();
					column2.step();
					/*修改鸟的位置(鸟的上抛)*/
					bird.step();
					/*实现鸟的振翅*/
					bird.fly();
				}
				/*碰撞地面*/
				if(bird.hit(ground)) {
					gameOver=true;
				}
				/*绑定鼠标监听事件*/
				MouseListener listener = new MouseAdapter() {
					/*鼠标按下*/
					public void mousePressed(MouseEvent e) {
						bird.flappy();
					}
				};
				/*将listener挂接到当前的面板（BirdGame）上*/
				addMouseListener(listener);
				/*计分逻辑*/				
				if(bird.x==column1.x || bird.x==column2.x) {
					score++;
				}
				/*重新绘制界面*/
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
			game.action();
		}
	}

⑤在`BirdGame`类的`paint`方法中，添加绘制游戏结束时，界面画面：

	package testbird;

	import java.awt.Color;
	import java.awt.Font;
	import java.awt.Graphics;
	import java.awt.Graphics2D;
	import java.awt.event.MouseAdapter;
	import java.awt.event.MouseEvent;
	import java.awt.event.MouseListener;
	import java.awt.image.BufferedImage;
	import javax.swing.JFrame;
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
		/*分数*/
		public int score;
		/*游戏结束*/
		public boolean gameOver;//游戏是否结束
		public BufferedImage gameOverImage;//游戏结束画面

		/*初始化BirdGame的属性变量*/
		public BirdGame() throws Exception {
			bird = new Bird();
			column1 = new Column(1);
			column2 = new Column(2);
			ground = new Ground();
			/*初始化游戏背景图片*/
			background = ImageIO.read(getClass().getResource("bg.png"));
			/*初始化游戏结束*/
			gameOver=false;
			gameOverImage= ImageIO.read(getClass().getResource("gameover.png"));
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
			/*绘制分数*/
			Font f=new Font(Font.SANS_SERIF, Font.BOLD, 40);
			g.setFont(f);
			g.drawString(""+score, 40, 60);
			g.setColor(Color.WHITE);
			g.drawString(""+score, 40-3, 60-3);
			/*游戏结束处理逻辑*/
			if(gameOver) {
				g.drawImage(gameOverImage, 0, 0, null);
			}

		}
	
		/*添加action方法,实现界面对象的移动效果*/
		public void action() throws Exception {
			while(true) {
				if(!gameOver) {
					/*修改地面位置*/
					ground.step();
					/*修改柱子位置*/
					column1.step();
					column2.step();
					/*修改鸟的位置(鸟的上抛)*/
					bird.step();
					/*实现鸟的振翅*/
					bird.fly();
				}
				/*碰撞地面*/
				if(bird.hit(ground)) {
					gameOver=true;
				}
				/*绑定鼠标监听事件*/
				MouseListener listener = new MouseAdapter() {
					/*鼠标按下*/
					public void mousePressed(MouseEvent e) {
						bird.flappy();
					}
				};
				/*将listener挂接到当前的面板（BirdGame）上*/
				addMouseListener(listener);
				/*计分逻辑*/				
				if(bird.x==column1.x || bird.x==column2.x) {
					score++;
				}
				/*重新绘制界面*/
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
			game.action();
		}
	}

> ### 步骤十五：实现鸟的碰撞检测：与柱子的碰撞检测） ###

这里鸟与柱子的碰撞情况有二种，第一种是与柱子周边的碰撞，第二种是与柱子缝隙间的碰撞。分别如下图：

与柱子周边（图22）：

![bird22.png](http://marshucheng1.github.io/assets/bird22.png)

与柱子缝隙（图23）：

![bird23.png](http://marshucheng1.github.io/assets/bird23.png)

从图22我们可以知道小鸟与柱子发生碰撞的x坐标计算如下：

`x1=column.x-width/2-size/2;x2=column.x+width/2+size/2;`当鸟的x坐标大于x1（x>x1）并且小于x2（x<x2）时与柱子发生碰撞。

从图23我们可以知道小鸟与柱子碰撞时，小鸟在柱子的缝隙中时y坐标计算如下：

`y1=column.x-gap/2+size/2;y2=column.x+gap/2-size/2;`
当鸟的y坐标大于y1（y>y1）并且小于y2（y<y2）时鸟在缝隙中。

通过上面的分析，在`Bird`类中添加`hit(Column)`方法，用于判断鸟是否碰撞柱子。

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
	
		/*碰撞地面检测方法*/
		public boolean hit(Ground ground) {
			boolean hit = y+size/2>ground.y;
			if(hit) {
				/*将鸟放置到地面上*/
				y=ground.y-size/2;
				/*使鸟摔倒在地面上有摔倒效果*/
				/*只是让鸟有一个动画效果,不清楚的可以注释掉仔细观察*/
				alpha= -3.14159265358979323/2;
			}
			return hit;
		}

		/* 碰撞柱子检测方法 */
		public boolean hit(Column column) {
			/* 先检测是否在柱子的范围以内(柱子外侧) */
			if (x > column.x - column.width / 2 - size / 2 && x < column.x + column.width / 2 + size / 2) {
				/* 检测是否在柱子缝隙中 */
				if (y > column.y - column.gap / 2 + size / 2 && 
						y < column.y + 	column.gap / 2 - size / 2) {
					return false;
				}
				return true;
			}
			return false;
		}

	}

在`BirdGame`类的`action`方法中，修改碰撞地面部分代码：

	package testbird;

	import java.awt.Color;
	import java.awt.Font;
	import java.awt.Graphics;
	import java.awt.Graphics2D;
	import java.awt.event.MouseAdapter;
	import java.awt.event.MouseEvent;
	import java.awt.event.MouseListener;
	import java.awt.image.BufferedImage;
	import javax.swing.JFrame;
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
		/*分数*/
		public int score;
		/*游戏结束*/
		public boolean gameOver;//游戏是否结束
		public BufferedImage gameOverImage;//游戏结束画面

		/*初始化BirdGame的属性变量*/
		public BirdGame() throws Exception {
			bird = new Bird();
			column1 = new Column(1);
			column2 = new Column(2);
			ground = new Ground();
			/*初始化游戏背景图片*/
			background = ImageIO.read(getClass().getResource("bg.png"));
			/*初始化游戏结束*/
			gameOver=false;
			gameOverImage= ImageIO.read(getClass().getResource("gameover.png"));
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
			/*绘制分数*/
			Font f=new Font(Font.SANS_SERIF, Font.BOLD, 40);
			g.setFont(f);
			g.drawString(""+score, 40, 60);
			g.setColor(Color.WHITE);
			g.drawString(""+score, 40-3, 60-3);
			/*游戏结束处理逻辑*/
			if(gameOver) {
				g.drawImage(gameOverImage, 0, 0, null);
			}

		}
	
		/*添加action方法,实现界面对象的移动效果*/
		public void action() throws Exception {
			while(true) {
				if(!gameOver) {
					/*修改地面位置*/
					ground.step();
					/*修改柱子位置*/
					column1.step();
					column2.step();
					/*修改鸟的位置(鸟的上抛)*/
					bird.step();
					/*实现鸟的振翅*/
					bird.fly();
				}
				/*碰撞地面和柱子*/
				if(bird.hit(ground) || bird.hit(column1) || bird.hit(column2)) {
					gameOver=true;
				}

				/*绑定鼠标监听事件*/
				MouseListener listener = new MouseAdapter() {
					/*鼠标按下*/
					public void mousePressed(MouseEvent e) {
						bird.flappy();
					}
				};
				/*将listener挂接到当前的面板（BirdGame）上*/
				addMouseListener(listener);
				/*计分逻辑*/				
				if(bird.x==column1.x || bird.x==column2.x) {
					score++;
				}
				/*重新绘制界面*/
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
			game.action();
		}
	}

> ### 步骤十六：实现游戏的开始、结束、以及重新开始 ###

我们把游戏归类为开始、结束、重新开始三个部分。定义为三种状态，`start`、`running`、`game_over`。

开始界面（图24）：

![bird24.png](http://marshucheng1.github.io/assets/bird24.png)

当游戏为`start`状态时，小鸟的翅膀是挥动的、地面是移动的。即在该状态时，调用`Bird`类的`fly`方法和`Ground`的`step`方法。

运行状态（图25）：

![bird25.png](http://marshucheng1.github.io/assets/bird25.png)

当游戏为`running`时，小鸟的翅膀是挥舞的、小鸟是上下移动的、地面是移动的，柱子是移动的。

即在该状态时，调用`Bird`的`fly`方法、`Bird`的`step`方法、`Ground`的`step`方法、`Column`的`step`方法、`Column`的`step`方法。同时也要进行碰撞检测。

结束状态（图26）：

![bird26.png](http://marshucheng1.github.io/assets/bird26.png)

当游戏为`game_over`时，只显示游戏结束界面，并且点击界面可以重新开始游戏。

介绍完游戏的运行状态后，我们来了解下各个状态间的相互转换：

鼠标按下时，如果游戏为`game_over`状态，则将下列信息进行重新初始化，并把状态更改为`start`，即实现了重新开始。

	bird = new Bird();
	column1 = new Column(1);
	column2 = new Column(2);
	ground = new Ground();
	score=0;
	state=START;

鼠标按下时，如果游戏为`start`状态，则将状态更新为`running`

	state=RUNNING;

最后，当鼠标按下时如果当前游戏状态为`running`，则使小鸟飞扬，即调用`Bird`类的`flappy`方法。

了解完游戏中各个状态的相互转换，我们就可以一步步的实现游戏的开始、结束、重新开始了。

1.在`BirdGame`类中定义游戏状态属性，增加游戏开始界面属性，并注释掉原来的`gameOver`属性。
构造方法中的初始化也注释掉。

**这里注释掉代码后，还会有些错误。不用理会，后面会一步步解决。**

	package testbird;

	import java.awt.Color;
	import java.awt.Font;
	import java.awt.Graphics;
	import java.awt.Graphics2D;
	import java.awt.event.MouseAdapter;
	import java.awt.event.MouseEvent;
	import java.awt.event.MouseListener;
	import java.awt.image.BufferedImage;
	import javax.swing.JFrame;
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
		/*分数*/
		public int score;
		/*游戏结束*/
		//public boolean gameOver;//游戏是否结束
		/*游戏状态*/
		public int state;
		public static final int START=0;
		public static final int RUNNING=1;
		public static final int GAME_OVER=2;
		public BufferedImage startImage;//游戏开始画面

		public BufferedImage gameOverImage;//游戏结束画面

		/*初始化BirdGame的属性变量*/
		public BirdGame() throws Exception {
			bird = new Bird();
			column1 = new Column(1);
			column2 = new Column(2);
			ground = new Ground();
			/*初始化游戏背景图片*/
			background = ImageIO.read(getClass().getResource("bg.png"));
			/*初始化游戏结束*/
			//gameOver=false;
			gameOverImage= ImageIO.read(getClass().getResource("gameover.png"));
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
			/*绘制分数*/
			Font f=new Font(Font.SANS_SERIF, Font.BOLD, 40);
			g.setFont(f);
			g.drawString(""+score, 40, 60);
			g.setColor(Color.WHITE);
			g.drawString(""+score, 40-3, 60-3);
			/*游戏结束处理逻辑*/
			if(gameOver) {
				g.drawImage(gameOverImage, 0, 0, null);
			}

		}
	
		/*添加action方法,实现界面对象的移动效果*/
		public void action() throws Exception {
			while(true) {
				if(!gameOver) {
					/*修改地面位置*/
					ground.step();
					/*修改柱子位置*/
					column1.step();
					column2.step();
					/*修改鸟的位置(鸟的上抛)*/
					bird.step();
					/*实现鸟的振翅*/
					bird.fly();
				}
				/*碰撞地面和柱子*/
				if(bird.hit(ground) || bird.hit(column1) || bird.hit(column2)) {
					gameOver=true;
				}

				/*绑定鼠标监听事件*/
				MouseListener listener = new MouseAdapter() {
					/*鼠标按下*/
					public void mousePressed(MouseEvent e) {
						bird.flappy();
					}
				};
				/*将listener挂接到当前的面板（BirdGame）上*/
				addMouseListener(listener);
				/*计分逻辑*/				
				if(bird.x==column1.x || bird.x==column2.x) {
					score++;
				}
				/*重新绘制界面*/
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
			game.action();
		}
	}

2.在`BirdGame`类的构造方法中，初始化游戏状态及开始界面属性（游戏开始）。

	package testbird;

	import java.awt.Color;
	import java.awt.Font;
	import java.awt.Graphics;
	import java.awt.Graphics2D;
	import java.awt.event.MouseAdapter;
	import java.awt.event.MouseEvent;
	import java.awt.event.MouseListener;
	import java.awt.image.BufferedImage;
	import javax.swing.JFrame;
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
		/*分数*/
		public int score;
		/*游戏结束*/
		//public boolean gameOver;//游戏是否结束
		/*游戏状态*/
		public int state;
		public static final int START=0;
		public static final int RUNNING=1;
		public static final int GAME_OVER=2;
		public BufferedImage startImage;//游戏开始画面

		public BufferedImage gameOverImage;//游戏结束画面

		/*初始化BirdGame的属性变量*/
		public BirdGame() throws Exception {
			bird = new Bird();
			column1 = new Column(1);
			column2 = new Column(2);
			ground = new Ground();
			/*初始化游戏背景图片*/
			background = ImageIO.read(getClass().getResource("bg.png"));
			/*初始化游戏结束*/
			//gameOver=false;
			state=START;
			startImage= ImageIO.read(getClass().getResource("start.png"));

			gameOverImage= ImageIO.read(getClass().getResource("gameover.png"));
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
			/*绘制分数*/
			Font f=new Font(Font.SANS_SERIF, Font.BOLD, 40);
			g.setFont(f);
			g.drawString(""+score, 40, 60);
			g.setColor(Color.WHITE);
			g.drawString(""+score, 40-3, 60-3);
			/*游戏结束处理逻辑*/
			if(gameOver) {
				g.drawImage(gameOverImage, 0, 0, null);
			}

		}
	
		/*添加action方法,实现界面对象的移动效果*/
		public void action() throws Exception {
			while(true) {
				if(!gameOver) {
					/*修改地面位置*/
					ground.step();
					/*修改柱子位置*/
					column1.step();
					column2.step();
					/*修改鸟的位置(鸟的上抛)*/
					bird.step();
					/*实现鸟的振翅*/
					bird.fly();
				}
				/*碰撞地面和柱子*/
				if(bird.hit(ground) || bird.hit(column1) || bird.hit(column2)) {
					gameOver=true;
				}

				/*绑定鼠标监听事件*/
				MouseListener listener = new MouseAdapter() {
					/*鼠标按下*/
					public void mousePressed(MouseEvent e) {
						bird.flappy();
					}
				};
				/*将listener挂接到当前的面板（BirdGame）上*/
				addMouseListener(listener);
				/*计分逻辑*/				
				if(bird.x==column1.x || bird.x==column2.x) {
					score++;
				}
				/*重新绘制界面*/
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
			game.action();
		}
	}

3.在`BirdGame`类的`paint`方法中修改游戏结束代码。

	package testbird;

	import java.awt.Color;
	import java.awt.Font;
	import java.awt.Graphics;
	import java.awt.Graphics2D;
	import java.awt.event.MouseAdapter;
	import java.awt.event.MouseEvent;
	import java.awt.event.MouseListener;
	import java.awt.image.BufferedImage;
	import javax.swing.JFrame;
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
		/*分数*/
		public int score;
		/*游戏结束*/
		//public boolean gameOver;//游戏是否结束
		/*游戏状态*/
		public int state;
		public static final int START=0;
		public static final int RUNNING=1;
		public static final int GAME_OVER=2;
		public BufferedImage startImage;//游戏开始画面

		public BufferedImage gameOverImage;//游戏结束画面

		/*初始化BirdGame的属性变量*/
		public BirdGame() throws Exception {
			bird = new Bird();
			column1 = new Column(1);
			column2 = new Column(2);
			ground = new Ground();
			/*初始化游戏背景图片*/
			background = ImageIO.read(getClass().getResource("bg.png"));
			/*初始化游戏结束*/
			//gameOver=false;
			state=START;
			startImage= ImageIO.read(getClass().getResource("start.png"));

			gameOverImage= ImageIO.read(getClass().getResource("gameover.png"));
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
			/*绘制分数*/
			Font f=new Font(Font.SANS_SERIF, Font.BOLD, 40);
			g.setFont(f);
			g.drawString(""+score, 40, 60);
			g.setColor(Color.WHITE);
			g.drawString(""+score, 40-3, 60-3);
			/*游戏结束处理逻辑*/
			switch (state) {
				case GAME_OVER:
					g.drawImage(gameOverImage, 0, 0, null);
					break;
				case START:
					g.drawImage(startImage, 0, 0, null);
					break;
			}
		}
	
		/*添加action方法,实现界面对象的移动效果*/
		public void action() throws Exception {
			while(true) {
				if(!gameOver) {
					/*修改地面位置*/
					ground.step();
					/*修改柱子位置*/
					column1.step();
					column2.step();
					/*修改鸟的位置(鸟的上抛)*/
					bird.step();
					/*实现鸟的振翅*/
					bird.fly();
				}
				/*碰撞地面和柱子*/
				if(bird.hit(ground) || bird.hit(column1) || bird.hit(column2)) {
					gameOver=true;
				}

				/*绑定鼠标监听事件*/
				MouseListener listener = new MouseAdapter() {
					/*鼠标按下*/
					public void mousePressed(MouseEvent e) {
						bird.flappy();
					}
				};
				/*将listener挂接到当前的面板（BirdGame）上*/
				addMouseListener(listener);
				/*计分逻辑*/				
				if(bird.x==column1.x || bird.x==column2.x) {
					score++;
				}
				/*重新绘制界面*/
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
			game.action();
		}
	}

4.在`BirdGame`类的`action`方法中修改鼠标监听处理逻辑。

	package testbird;

	import java.awt.Color;
	import java.awt.Font;
	import java.awt.Graphics;
	import java.awt.Graphics2D;
	import java.awt.event.MouseAdapter;
	import java.awt.event.MouseEvent;
	import java.awt.event.MouseListener;
	import java.awt.image.BufferedImage;
	import javax.swing.JFrame;
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
		/*分数*/
		public int score;
		/*游戏结束*/
		//public boolean gameOver;//游戏是否结束
		/*游戏状态*/
		public int state;
		public static final int START=0;
		public static final int RUNNING=1;
		public static final int GAME_OVER=2;
		public BufferedImage startImage;//游戏开始画面

		public BufferedImage gameOverImage;//游戏结束画面

		/*初始化BirdGame的属性变量*/
		public BirdGame() throws Exception {
			bird = new Bird();
			column1 = new Column(1);
			column2 = new Column(2);
			ground = new Ground();
			/*初始化游戏背景图片*/
			background = ImageIO.read(getClass().getResource("bg.png"));
			/*初始化游戏结束*/
			//gameOver=false;
			state=START;
			startImage= ImageIO.read(getClass().getResource("start.png"));

			gameOverImage= ImageIO.read(getClass().getResource("gameover.png"));
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
			/*绘制分数*/
			Font f=new Font(Font.SANS_SERIF, Font.BOLD, 40);
			g.setFont(f);
			g.drawString(""+score, 40, 60);
			g.setColor(Color.WHITE);
			g.drawString(""+score, 40-3, 60-3);
			/*游戏结束处理逻辑*/
			switch (state) {
			case GAME_OVER:
				g.drawImage(gameOverImage, 0, 0, null);
				break;
			case START:
				g.drawImage(startImage, 0, 0, null);
				break;
			}
		}
	
		/*添加action方法,实现界面对象的移动效果*/
		public void action() throws Exception {
			while(true) {
				if(!gameOver) {
					/*修改地面位置*/
					ground.step();
					/*修改柱子位置*/
					column1.step();
					column2.step();
					/*修改鸟的位置(鸟的上抛)*/
					bird.step();
					/*实现鸟的振翅*/
					bird.fly();
				}
				/*碰撞地面和柱子*/
				if(bird.hit(ground) || bird.hit(column1) || bird.hit(column2)) {
					gameOver=true;
				}

				/*绑定鼠标监听事件*/
				MouseListener listener = new MouseAdapter() {
					/*鼠标按下*/
					public void mousePressed(MouseEvent e) {
						//bird.flappy();
						try {
							switch (state) {
							case GAME_OVER:
								bird=new Bird();
								column1=new Column(1);
								column2=new Column(2);
								score=0;
								state=START;
								break;
							case START:
								state=RUNNING;
								break;
							case RUNNING:
								//鸟向上飞扬
								bird.flappy();
							}
						} catch (Exception e2) {
							e2.printStackTrace();
						}
					}
				};
				/*将listener挂接到当前的面板（BirdGame）上*/
				addMouseListener(listener);
				/*计分逻辑*/				
				if(bird.x==column1.x || bird.x==column2.x) {
					score++;
				}
				/*重新绘制界面*/
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
			game.action();
		}
	}

5.在`BirdGame`类的`action`方法中修改`while`循环部分的处理逻辑，以及修改碰撞部分代码。

	package testbird;

	import java.awt.Color;
	import java.awt.Font;
	import java.awt.Graphics;
	import java.awt.Graphics2D;
	import java.awt.event.MouseAdapter;
	import java.awt.event.MouseEvent;
	import java.awt.event.MouseListener;
	import java.awt.image.BufferedImage;
	import javax.swing.JFrame;
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
		/*分数*/
		public int score;
		/*游戏结束*/
		//public boolean gameOver;//游戏是否结束
		/*游戏状态*/
		public int state;
		public static final int START=0;
		public static final int RUNNING=1;
		public static final int GAME_OVER=2;
		public BufferedImage startImage;//游戏开始画面

		public BufferedImage gameOverImage;//游戏结束画面

		/*初始化BirdGame的属性变量*/
		public BirdGame() throws Exception {
			bird = new Bird();
			column1 = new Column(1);
			column2 = new Column(2);
			ground = new Ground();
			/*初始化游戏背景图片*/
			background = ImageIO.read(getClass().getResource("bg.png"));
			/*初始化游戏结束*/
			//gameOver=false;
			state=START;
			startImage= ImageIO.read(getClass().getResource("start.png"));

			gameOverImage= ImageIO.read(getClass().getResource("gameover.png"));
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
			/*绘制分数*/
			Font f=new Font(Font.SANS_SERIF, Font.BOLD, 40);
			g.setFont(f);
			g.drawString(""+score, 40, 60);
			g.setColor(Color.WHITE);
			g.drawString(""+score, 40-3, 60-3);
			/*游戏结束处理逻辑*/
			switch (state) {
			case GAME_OVER:
				g.drawImage(gameOverImage, 0, 0, null);
				break;
			case START:
				g.drawImage(startImage, 0, 0, null);
				break;
			}
		}
	
		/*添加action方法,实现界面对象的移动效果*/
		public void action() throws Exception {
			while(true) {
				switch (state) {
				case START:
					/* 实现鸟的振翅 */
					bird.fly();
					/* 修改地面位置 */
					ground.step();
					break;
				case RUNNING:
					/* 修改柱子位置 */
					column1.step();
					column2.step();
					/* 修改鸟的位置（鸟的上抛） */
					bird.step();
					/* 实现鸟的振翅 */
					bird.fly();
					/* 修改地面位置 */
					ground.step();
				}
				/* 碰撞地面和柱子 */
				if (bird.hit(ground) || bird.hit(column1) || bird.hit(column2)) {
					state = GAME_OVER;
				}

				/*绑定鼠标监听事件*/
				MouseListener listener = new MouseAdapter() {
					/*鼠标按下*/
					public void mousePressed(MouseEvent e) {
						//bird.flappy();
						try {
							switch (state) {
							case GAME_OVER:
								bird=new Bird();
								column1=new Column(1);
								column2=new Column(2);
								score=0;
								state=START;
								break;
							case START:
								state=RUNNING;
								break;
							case RUNNING:
								//鸟向上飞扬
								bird.flappy();
							}
						} catch (Exception e2) {
							e2.printStackTrace();
						}
					}
				};
				/*将listener挂接到当前的面板（BirdGame）上*/
				addMouseListener(listener);
				/*计分逻辑*/				
				if(bird.x==column1.x || bird.x==column2.x) {
					score++;
				}
				/*重新绘制界面*/
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
			game.action();
		}
	}

至此，整个游戏的开发就完成了。我们已经成功实现了一个飞扬小鸟的游戏！

这里我们还有一个重要的问题没有处理。前面我们在调用`BirdGame`类中的`action`方法时。

	/* 设置窗口显示可见 */
	frame.setVisible(true);
	game.action();

我们将`game.action();`的调用放到了最后，如果不这样的话，是看不到画面的（如下，不妨试一下）。

	game.action();
	/* 设置窗口显示可见 */
	frame.setVisible(true);

因为，我们当前的程序只有一个主线程main，如果不在最后再调用`action`方法，`main`线程会默认顺序执行，然后进入死循环状态，无法执行后面窗口显示部分的内容。

现在我们可以改变一下我们的代码，我们可以在`action`方法中，构造一个匿名内部类形式的线程。

这样我们就可以在任意位置调用`game.action`，并且不会影响游戏的正常运行。因为此时，我们的游戏
有两条线程参与。

具体代码如下：

	package testbird;

	import java.awt.Color;
	import java.awt.Font;
	import java.awt.Graphics;
	import java.awt.Graphics2D;
	import java.awt.event.MouseAdapter;
	import java.awt.event.MouseEvent;
	import java.awt.event.MouseListener;
	import java.awt.image.BufferedImage;
	import javax.swing.JFrame;
	import javax.swing.JPanel;
	import javax.imageio.ImageIO;

	public class BirdGame extends JPanel {
		/* 在界面添加鸟 */
		public Bird bird;
		/* 在界面添加两个柱子的左边柱子 */
		public Column column1;
		/* 在界面添加两个柱子的右边柱子 */
		public Column column2;
		/* 在界面添加地面 */
		public Ground ground;
		/* 主界面背景图片 */
		public BufferedImage background;
		/* 分数 */
		public int score;
		/* 游戏结束 */
		// public boolean gameOver;
		/* 游戏状态 */
		public int state;
		public static final int START = 0;
		public static final int RUNNING = 1;
		public static final int GAME_OVER = 2;
		public BufferedImage startImage;

		public BufferedImage gameOverImage;

		/* 初始化BirdGame的属性变量 */
		public BirdGame() throws Exception {
			bird = new Bird();
			column1 = new Column(1);
			column2 = new Column(2);
			ground = new Ground();
			/* 初始化游戏背景图片 */
			background = ImageIO.read(getClass().getResource("bg.png"));
			/* 初始化游戏结束 */
			// gameOver=false;
			state = START;
			startImage = ImageIO.read(getClass().getResource("start.png"));

			gameOverImage = ImageIO.read(getClass().getResource("gameover.png"));

		}

		/* 重写paint方法,绘制界面 */
		public void paint(Graphics g) {
			/* 绘制背景 */
			g.drawImage(background, 0, 0, null);
			/* 绘制柱子 */
			g.drawImage(column1.image, column1.x - column1.width / 2, column1.y - column1.height / 2, null);
			g.drawImage(column2.image, column2.x - column2.width / 2, column2.y - column2.height / 2, null);
			/* 绘制地面 */
			g.drawImage(ground.image, ground.x, ground.y, null);
			/* 实现鸟的倾角动画 */
			Graphics2D g2 = (Graphics2D) g;
			g2.rotate(-bird.alpha, bird.x, bird.y);
			/* 绘制鸟 */
			g.drawImage(bird.image, bird.x - bird.width / 2, bird.y - bird.height / 2, null);
			g2.rotate(bird.alpha, bird.x, bird.y);
			/* 绘制分数 */
			Font f = new Font(Font.SANS_SERIF, Font.BOLD, 40);
			g.setFont(f);
			g.drawString("" + score, 40, 60);
			g.setColor(Color.WHITE);
			g.drawString("" + score, 40 - 3, 60 - 3);
			/* 游戏结束处理逻辑 */
			switch (state) {
			case GAME_OVER:
				g.drawImage(gameOverImage, 0, 0, null);
				break;
			case START:
				g.drawImage(startImage, 0, 0, null);
				break;
			}
		}

		/* 添加action方法,实现界面对象的移动效果 */
		public void action() throws Exception {
			new Thread() {
				public void run() {
					while (true) {
						switch (state) {
						case START:
							/* 实现鸟的振翅 */
							bird.fly();
							/* 修改地面位置 */
							ground.step();
							break;
						case RUNNING:
							/* 修改柱子位置 */
							column1.step();
							column2.step();
							/* 修改鸟的位置（鸟的上抛） */
							bird.step();
							/* 实现鸟的振翅 */
							bird.fly();
							/* 修改地面位置 */
							ground.step();
						}
						/* 碰撞地面和柱子 */
						if (bird.hit(ground) || bird.hit(column1) || bird.hit(column2)) {
							state = GAME_OVER;
						}
						/* 绑定鼠标监听事件 */
						MouseListener listener = new MouseAdapter() {
							/* 鼠标按下 */
							public void mousePressed(MouseEvent e) {
								// bird.flappy();
								try {
									switch (state) {
									case GAME_OVER:
										bird = new Bird();
										column1 = new Column(1);
										column2 = new Column(2);
										score = 0;
										state = START;
										break;
									case START:
										state = RUNNING;
										break;
									case RUNNING:
										// 鸟向上飞扬
										bird.flappy();
									}
								} catch (Exception e2) {
									e2.printStackTrace();
								}
							}
						};
						/* 将listener挂接到当前的面板（BirdGame）上 */
						addMouseListener(listener);
						/* 计分逻辑 */
						if (bird.x == column1.x || bird.x == column2.x) {
							score++;
						}
						/* 重新绘制界面 */
						repaint();
						/* 设置屏幕的刷新率为1秒30次 */
						try {
							Thread.sleep(1000 / 30);
						} catch (InterruptedException e1) {
							e1.printStackTrace();
						}
					}
				}
			}.start();
		}

		/* 启动软件的方法 */
		public static void main(String[] args) throws Exception {
			/* 初始化一个操作窗口 */
			JFrame frame = new JFrame();
			/* 初始化游戏主面板 */
			BirdGame game = new BirdGame();
			/* 将游戏主界面面板添加到操作窗口中 */
			frame.add(game);
			/* 设置主界面宽、高 */
			frame.setSize(432, 644);
			/* 让主界面在电脑屏幕居中显示 */
			frame.setLocationRelativeTo(null);
			/* 设置窗口关闭方式,点窗口关闭x号,java程序也自动结束运行 */
			frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

			/* 设置窗口显示可见 */
			// game.action();
			frame.setVisible(true);
			game.action();
		}
	}
