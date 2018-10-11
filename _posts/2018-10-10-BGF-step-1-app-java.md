---
layout: post
title: BGF Step 1 - App.java
categories: [java, gamedev]
tags: [gameloop, core, framework, basic]
excerpt_separator: <!--more-->
---

This post is all about starting a new java application, for a Desktop usage, and in the context of a game development.
<!--more-->

# BGF step 1: App.java

We are going to start by design a main class for our game, then add some useful classes to manage game objects, some service like collision, and others things like states and so on...

## let's talk about

### What we want to achieve

The application I am targeting at is a simple java application with a `main(String[])`  method to start all. This **B**asic **G**ame **F**ramework must provide all the basics to start coding a simple 2D platform game.

### What is awaited

- a working GameLoop,
- a basic graphical object to manage sprite and decors on the game display
- an easy way to build simple UI (menu, items, etc...)
- a Layered rendering pipe to manage easily multiple graphics layers.
- a collision system with response management
- some very basic physic (position, speed, acceleration, elasticity and friction).
- and a State manager to implements various game play and screens in the game, without coding the state initialization/switching.

Ok, the "Liste à la Prévert" is now ready, let's dive in the solution we want to code.

## Code and Intention

### App.java

All will start with the `App.java` file. This main class will extends the `java.awt.JPanel`  to be integrated in a `java.awt.JFrame`, the default java windowing system.

A good class must contains some attributes, and here we will have a bunch of those old pals :

> ***INFO***
> *the App class will implements some commons java interface, `Runnable` to assume a Thread behavior, and `KeyListener` to capture and process key events.*

```java
public class App extends JPanel
   implements Runnable, KeyListener{

  // a title for the future window
  private String title="BGF";

  // a flag to manage the exit game request
  private boolean exit = false;

  // some globals graphic parameters
  private static int WIDTH=320;
  private static int HEIGHT=320;
  private static float SCALE=2;
  
  // a rendering buffer for all graphic things
  private BufferedImage buffer;
  
  // a global reference to the graphic API to be used
  private Graphics2D g;
  private Rectangle viewport;
  private static int FPS = 60;
  private int timeFrame=(1000/FPS);
  private int realFPS = 0;

  // a status to manage debug display in the game.
  private static int DEBUG=0;

  // a thread to delegate processing.
  private Thread thread;
```

A good constructor with some parameters: the first step in the constructor is to parse the java command line arguments and try to extract some parameters. Then start instantiate things.

```java
  public App(String[] args){
    // parse the java CLI
    parseArgs(args);
    
    // start the Thread
    thread = new Thread(this);
    
    // set the grahics buffer to draw things
    buffer =  new  BufferedImage(
      WIDTH, HEIGHT, 
      BufferedImage.TYPE_INT_ARGB);
    
    // get the redenring API  
    g = (Graphics2D) buffer.getGraphics();
    
    // set the `boudingbox`for the gameplay area
    viewport =  new  Rectangle(
      buffer.getWidth(), 
      buffer.getHeight());  
  }
```

> ***INFO***
> *The default `buffer` is a RBG + alpha one to manage transparency. 
> The `Graphics2D` API is initialized for the all game graphic operation. 
> The `viewport` is a bounding box, it will be used for collision and constrain object to stay in the display area.*
> The `thread` is a `Runnable` instance for our App. It's a java Job.

Then all the internal kitchen to manage game looping :

```java
 
  // Initialize all the game objects and services !
  public void initialize(){}
 
  // where to manage any user input
  public void input(){}
 
  // to process and update the game mechanics
  public void update(long dt){}
 
  // where to render all those beautiful graphics
  public void render(Graphics2D g){}
 
  // free all resources.
  public void dispose(){}
  
  // And here is where all begins !
  public void run(){

    // initialize the game
    initialize();
    
    // start measuring the time.
    long current = System.currentMillis();
    long previous = current;
    long elasped = 0;
    
    // the intermediate variable for computing realFPS
    long  cumulation  =  0, frames =  0;
    
    // all the magic of the loop.
    while(!exit){
      current = System.currentMillis();
      //manage user input
      input();
      // update game mechanics
      update(dt);
      // render the game objects
      render(g);
      
      elapsed = System.currentMillis()-current;
      
      // compute the real frame per second counter
      cumulation += elapsed;
      frames++;
      if (cumulation >  1000) {
        cumulation =  0;
        realFPS = frames;
        frames =  0;
      }
      
      // compute if we have time for a break
      if(elasped<timeFrame){
        try{
          Thread.sleep(timeFrame-elapsed);
        }catch(InterruptedException ie){
          System.err.println(
              "Unable to wait for timeframing");
        }
      }
      previous = current;
    }
    
    // free all allocated resources.
    dispose();
    
    // quite quit the app.
    System.exit(0);
  }
```
We implement now the `KeyListener` interface methods :
```java
  public void keyPressed(KeyEvent e){}
  public void keyReleased(KeyEvent e){}
  public void keyTyped(KeyEvent e){}
```
> ***INFO***
> *About the key processing, this mainly be a game matter, and not a framework matter, but we will implement later basic common processing.*
 
A parsing method to extract CLI parameters from the java command to preset some internal values:

if a Start my java from the command line below:
```bash
$> java -jar App-0.0.1-SNAPSHOT.jr -Dw=320 -Dh=200 -Dd=1 -Ds=2
```
we will got a String array with :
```java
String[]("w=320","h=200","d=1","s=2")
```
This will be parsed by the following code:

```java
  public void parseArgs(String[] args){
    if(args.length>0){
    for(String arg:args){
      if(arg.contains("=")){
        String[]values = arg.split('=');
        switch(values[0].toLowerCase()){
          case "w":
            WIDTH = Integer.parseInt(values[1]);
            break;
          case "h":
            HEIGHT = Integer.parseInt(values[1]);
            break;
          case "s":
            SCALE = Float.parseFloat(values[1]);
            break;
          case "d":
            DEBUG = Integer.parseInt(values[1]);
            break; 
          case "f":
            FPS = Integer.parseInt(values[1]);
            break;
        }
      }
    }
  }
```
The available parameters are :
| Param |      | Description                                  |
|:-----:|:----:|:---------------------------------------------|
| w     | 320  | set the window width                         |
| h     | 200  | set the window height                        |
| s     | 2    | define the pixel scaling to size the screen  |
| d     | 0    | set the debug display level from 0=off to 5  |
| f     | 60   | set the Frame Per Second rate                |

> ***INFO***
> *You will be able to add any kind of parameters for your own usage, adding new cases to the switch.*

And the entry point of the story, the main method, where we create the App, create a window (JFrame) and linked them all together.

```java
  public static main(String[] args){
    
    // Create a instance of our app 
    App app = new App(args);
    
    // Get the required diemnsion of the window
    Dimension dim = new Dimension(
        (int)(App.WIDTH*App.SCALE),
        (int)(App.HEGHT*App.SCALE)

    // Create and display the corresponding window
    JFrame frame = new JFrame("myGame");
    frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
    frame.setContentPane(app);
    frame.setSize(dim);
    frame.setMaximumSize(dim);
    frame.setMinimumSize(dim);
    frame.setPreferredSize(dim);
    frame.setLayout(new BorderLayout());
    frame.setResizable(false);
    frame.pack();
    frame.setVisible(true);

    // Start this game !
    app.run();
  }
}
```
OK.
Why having a try at running it ?

```bash
$> java -jar App-0.0.1-SNAPSHOT.jar
```
You would obtain a single window named "myGame", and that all.

> ***INFO***
> *The project on [github.com](https://github.com/snapgames/basic-game-framework "open the git project on github.com") is a maven project, so you will be able to start the project with the following command line:
> ```bash
> $> mvn clean install exec:java
> ```

### Some references

- The git repository for this project is at [github.com](https://github.com/snapgames/basic-game-framework "open the git project on github.com") 
- It is automatically build by [Travis-CI.org](https://travis-ci.org "discover this incredible build pipeline")
- it's code quality is surveyed by [codacity.com](https://app.codacy.com/project/mcgivrer/basic-game-framework/dashboard "open the ultimate code quality covering tool")
- and the license of dependencies is verified by [fossa.io](https://app.fossa.io/projects/git%2Bgithub.com%2FSnapGames%2Fbasic-game-framework/refs/branch/develop/ "open the Licensing manager")


@McGivrer#2018-OCT-11#

> Written with [StackEdit](https://stackedit.io/).
