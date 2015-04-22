title: "Create a Live Aquarium Wallpaper in Android"
date: 2010-12-27 12:56:03
tags:
---

![Final Result 1](http://cdn.rajeeshcv.com/images/2010/12/image_thumb.png)
![Final Result 2](http://cdn.rajeeshcv.com/images/2010/12/image_thumb1.png)

Few weeks ago I started learning Android programming, so this article is an outcome of that out-side office study :). Here I will be explaining – how to create a live wallpaper which looks like an aquarium with fishes swimming across the screen. The fish animation is done using sprite technique.

Fish sprite used here is from a [code project article](http://www.codeproject.com/KB/GDI-plus/LovelyGoldFishDeskPet.aspx)

Lets get started...

Creating a new Android project in eclipse (I am not familiar with any other IDEs for Android development :) ). Now create a class for your live wallpaper service, I called it as `AquariumWallpaperService`, then instantiate the `AquariumWallpaperEngine` which is responsible for creating the actual `Aquarium` class which does all the rendering logic. It also controls the flow of Aquarium based Surface callbacks Below is the code for  `AquariumWallpaperService`

```java
public class AquariumWallpaperService extends WallpaperService {
  @Override
  public Engine onCreateEngine() {
      return new AquariumWallpaperEngine();
  }

  class AquariumWallpaperEngine extends Engine{

      private Aquarium _aquarium;

      public AquariumWallpaperEngine() {
          this._aquarium = new Aquarium();
          this._aquarium.initialize(getBaseContext(), getSurfaceHolder());
      }

      @Override
      public void onVisibilityChanged(boolean visible) {
          if(visible){
              this._aquarium.render();
          }
      }

      @Override
      public void onSurfaceChanged(SurfaceHolder holder, int format,
              int width, int height) {
          super.onSurfaceChanged(holder, format, width, height);
      }

      @Override
      public void onSurfaceCreated(SurfaceHolder holder) {
          super.onSurfaceCreated(holder);
          this._aquarium.start();
      }

      @Override
      public void onSurfaceDestroyed(SurfaceHolder holder) {
          super.onSurfaceDestroyed(holder);
          this._aquarium.stop();
      }
  }
}
```

`Aquarium` class wraps all the rendering logic, as well as creating the fishes. This also starts a thread which is responsible for updating the view.

```java
public class Aquarium {

    private AquariumThread _aquariumThread;
    private SurfaceHolder _surfaceHolder;
    private ArrayList<Renderable> _fishes;
    private Bitmap _backgroundImage;
    private Context _context;

    public void render(){
        Canvas canvas = null;
        try{

            canvas = this._surfaceHolder.lockCanvas(null);
            synchronized (this._surfaceHolder) {
                this.onDraw(canvas);
            }

        }finally{
            if(canvas != null){
                this._surfaceHolder.unlockCanvasAndPost(canvas);
            }
        }
    }

    protected void onDraw(Canvas canvas) {
        this.renderBackGround(canvas);
        for (Renderable renderable : this._fishes) {
            renderable.render(canvas);
        }
    };

    public void start(){
        this._aquariumThread.switchOn();
    }

    public void stop(){
        boolean retry = true;
        this._aquariumThread.switchOff();
        while (retry) {
            try {
                this._aquariumThread.join();
                retry = false;
            } catch (InterruptedException e) {
                // we will try it again and again...
            }
        }
    }

    public int getLeft() {
        return 0;
    }

    public int getRight() {
        return this._backgroundImage.getWidth();
    }

    public void initialize(Context context, SurfaceHolder surfaceHolder) {
        this._aquariumThread = new AquariumThread(this);
        this._surfaceHolder = surfaceHolder;
        this._fishes = new ArrayList<Renderable>();
        this._context = context;
        BitmapFactory.Options options = new BitmapFactory.Options();
        options.inPurgeable = true;
        this._backgroundImage = BitmapFactory.decodeResource(context.getResources(), com.plugai.android.livewallpapers.R.drawable.aquarium, options);
        this.addFishes();
    }

    private void addFishes() {
        Point startPoint = new Point(100, 100);
        this._fishes.add(new ClownFish(this._context, this, startPoint, 90));
        Point startPoint1 = new Point(100, 300);
        this._fishes.add(new ClownFish(this._context, this, startPoint1, 50));

        Point startPoint2 = new Point(200, 200);
        this._fishes.add(new ClownFish(this._context, this, startPoint2, 15));
    }

    private void renderBackGround(Canvas canvas)
    {
        canvas.drawBitmap(this._backgroundImage, 0, 0, null);
    }
}
```

Here is the code for `AquariumThread` class

```java
public class AquariumThread extends Thread {

    private Aquarium _aquarium;
    private boolean _running;

    public AquariumThread(Aquarium aquarium) {
        this._aquarium = aquarium;
    }

    public void switchOn(){
        this._running = true;
        this.start();
    }

    public void pause(){
        this._running = false;
        synchronized(this){
            this.notify();
        }
    }

    public void switchOff(){
        this._running = false;
        synchronized(this){
            this.notify();
        }
    }

    @Override
    public void run() {
        while(this._running){
            this._aquarium.render();
        }
    }
}
```

All the *renderable* object in the aquarium must implement an interface `Renderable`, which has got one single method called `render(…)`

```java
package com.plugai.android.livewallpapers;

import android.graphics.Canvas;

public interface Renderable {
    void render(Canvas canvas);
}
```

This interface helps to render an object other than a fish, like plants etc... in future. I have created another abstract class which has common functionalities like changing the position and direction of a fish after a particular interval and called it as `AquaticAnimal`, this is because I could create specific fishes which just differ by it’s look by extending from this class.

```java
public abstract class AquaticAnimal implements Renderable {

    private static int MAX_SPEED = 100;
    private Context _context;
    private Aquarium _aquarium;
    private FishSprite _leftSprite;
    private FishSprite _rightSprite;

    private int _direction = -1;
    private int _speedFraction;
    private long _previousTime;

    public AquaticAnimal(Context context, Aquarium aquarium){
        this._context = context;
        this._aquarium = aquarium;
    }

    protected void initialize(Bitmap leftBitmap, Bitmap rightBitmap, int fps, int totalFrames, Point startPoint, int speed){
        this._leftSprite = new FishSprite(leftBitmap, fps, totalFrames, startPoint);
        this._rightSprite = new FishSprite(rightBitmap, fps, totalFrames, startPoint);
        this._speedFraction = (MAX_SPEED / speed) * 10;
    }

    private FishSprite getSprite(){
        if(this._direction < 0){
            return this._leftSprite;
        }
        return this._rightSprite;
    }

    public int getDirection(){
        FishSprite sprite = this.getSprite();
        int xPos = sprite.getXPos();
        if(this._direction < 0){
            xPos += sprite.getWidth();
        }
        if(xPos < this._aquarium.getLeft()){
            this._direction = 1;
        }else if(xPos > this._aquarium.getRight()){
            this._direction = -1;
        }else{
            // Do nothing
        }

        return this._direction;
    }

    public Context getContext(){
        return this._context;
    }

    public Aquarium getAquarium(){
        return this._aquarium;
    }

    @Override
    public void render(Canvas canvas){
        long currentTime = System.currentTimeMillis();
        this.getSprite().render(canvas, currentTime);
        this.swim(currentTime);
    }

    public void swim(long currentTime){
        long diff = currentTime - this._previousTime;
        if(diff > this._speedFraction){
            int currentX = this.getSprite().getXPos();
            this.getSprite().setXPos(currentX + this.getDirection());
            this._previousTime = currentTime;
        }
    }

}
```

The sprite animation is moved into a specific class `FishSprite`

```java
public class FishSprite {

    /**
     * Private fields
     */
    private Bitmap _currentSpriteBitmap;
    private Rect _drawRect;
    private int _fps;
    private int _noOfFrames;
    private int _currentFrame;
    private long _timer;
    private int _spriteWidth;
    private int _spriteHeight;
    private Point _position;

    public FishSprite(Bitmap spriteBitmap, int fps, int frameCount, Point startPoint) {

        this.initialize();

        this._position = startPoint;
        this._currentSpriteBitmap = spriteBitmap;
        this._spriteHeight = spriteBitmap.getHeight();
        this._spriteWidth = spriteBitmap.getWidth() / frameCount;
        this._drawRect = new Rect(0,0, this._spriteWidth, this._spriteHeight);
        this._fps = 1000 / fps;
        this._noOfFrames = frameCount;
    }

    private void initialize() {
        this._drawRect = new Rect(0,0,0,0);
        this._timer = 0;
        this._currentFrame = 0;
    }

    private void Update(long currentTime) {
        if(currentTime > this._timer + this._fps ) {
            this._timer = currentTime;
            this._currentFrame +=1;

            if(this._currentFrame >= this._noOfFrames) {
                this._currentFrame = 0;
            }
        }

        this._drawRect.left = this._currentFrame * this._spriteWidth;
        this._drawRect.right = this._drawRect.left + this._spriteWidth;
    }

    public void render(Canvas canvas, long currentTime) {

        this.Update(currentTime);

        Rect dest = new Rect(getXPos(), getYPos(), getXPos() + this._spriteWidth,
                        getYPos() + this._spriteHeight);

        canvas.drawBitmap(this._currentSpriteBitmap, this._drawRect, dest, null);
    }

    public Point getPosition() {
        return _position;
    }

    public void setPosition(Point position) {
        this._position = position;
    }

    public int getYPos() {
        return this._position.y;
    }

    public int getXPos() {
        return this._position.x;
    }

    public void setYPos(int y) {
        this._position.y = y;
    }

    public void setXPos(int x) {
        this._position.x = x;
    }

    public int getWidth(){
        return this._spriteWidth;
    }

    public int getHeight(){
        return this._spriteHeight;
    }

}
```

Now to the final bit, that is creating a fish. you might have noticed that I have created an object of `ClownFish `in the Aquarium class. `ClownFish` is just a derived class from `AquaticAnimal`, with a specific sprite image. So if you have sprite images for a shark, you could simple extend a new class for a shark with that image.

```java
public class ClownFish extends AquaticAnimal {
    private static final int TOTAL_FRAMES_IN_SPRITE = 20;
    private static final int CLOWN_FISH_FPS = 20;

    public ClownFish(Context context, Aquarium aquarium,  Point startPoint, int speed){
        super(context, aquarium);
        BitmapFactory.Options options = new BitmapFactory.Options();
        options.inPurgeable = true;
        Bitmap leftBitmap = BitmapFactory.decodeResource(getContext().getResources(), com.plugai.android.livewallpapers.R.drawable.left, options);
        BitmapFactory.Options options1 = new BitmapFactory.Options();
        options1.inPurgeable = true;
        Bitmap rightBitmap = BitmapFactory.decodeResource(getContext().getResources(), com.plugai.android.livewallpapers.R.drawable.right, options1);
        this.initialize(leftBitmap, rightBitmap, CLOWN_FISH_FPS, TOTAL_FRAMES_IN_SPRITE, startPoint, speed);
    }

    public void render(Canvas canvas){
        super.render(canvas);
    }
}
```

Please feel free to go through the code, If I haven’t explained clearly. Hope this article helped some way or other.

**Happy new year!!!**

> Full source code is available [here](https://github.com/cvrajeesh/LiveAquariumWallpaper)
