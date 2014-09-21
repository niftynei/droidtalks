
# __*getBounds()*__


# [fit] The story of Drawables and their View masters

#### __*Jamie Huson*__ + __*Lisa Neigut*__ | __20 Sept 2014__

^ Hi, Good morning. The title of this talk is getBounds(): the story of drawables and their view masters. 

---


![right|fit] (lisa.jpg)

![left|fit] (jamie.jpg)


^ To start, I’d like to introduce myself and my collaborator and co-worker Jamie Huson. We both work at Etsy on the Android apps. Etsy the online market for handmade and vintage goods that we make together.

^ Jamie wasn’t able to make it out to New York to present with me today, so it’s my pleasure to be presenting on behalf of both of us.


—

#[fit]Drawables!


^ So let’s talk about Drawables.  The inspiration from this talk comes from Cyril Mottier’s excellent Mastering Android Drawables talk at Devoxx 2013. Today we’ll pick up a bit from where Cyril left off and take a deep dive into the specifics of the drawables APIs.  We’ll talk about how they relate to a View’s measurement/layout/draw cycles, and then go over where Drawables are headed for L.

^ So let’s get started.

---

# What is a Drawable?

---

### "A drawable is a positioned entity 
### that is to be drawn on a canvas." 

### __-Cyril Mottier__


^ Borrowing Cyril’s definition: A drawable is a positioned entity that is to be drawn on a canvas.

----

# What Makes Drawables Awesome:
- Drawing is fun!
- State Changes
- Total Separation of View Logic from Code
- Focused code
- XML

^ Ok, so what makes Drawables awesome? Pushing pixels to the screen is gratifying and fun! 

^ Drawables handle state changes for user feedback, essential to a good UX

^ Its focused Java code that separates the drawing logic from the other View logic

^ XML themes/styles combined with drawables means less code

^ Ok, so what Drawables do we get from the SDK?

---

## NinePatchDrawable

![right|80%] (nine_patch.png)

^ A drawable can be a 9 patch. That's a PNG that has a 1 pixel border specifying a stretchable region and padding.

---

## ShapeDrawable

![right|25%] (shape-sample.png)

^ It can be a shape that's specified in XML. It allows rectangle, line, oval, or a ring.

---

## StateListDrawable 

![right|25%] (state-list-drawable-sample.png)

^ Drawables are flexible and useful.  You can nest different drawables into each other.

^ It's a nice declarative approach to handling state changes.

---

## BitmapDrawable

![right|25%] (bitmap-sample.png)

^ BitmapDrawable is used everywhere and has some great options like repeating a pattern on a background

---

## LayerDrawable

![right|25%] (layer-list-sample.png)

^ Stack Drawables on top of each other in different ways!

---

#So many options…

![60%] (drawable_hierarchy.png)

^ The built in drawables are quite extensive.  So why extend it?

^ Let’s take a look at what you can do with custom drawables.

---

## BadgeDrawable
![fit|right] (badge_drawable.png)

^ Like Jesse Hendrickson’s BadgeDrawable class. It displays a number over 
a circle drawable.  This view is a single TextView, who’s right compoundDrawable has been
set to a BadgeDrawable instance. To update the number, we just change the number on the drawable.

——

##IconDrawable
![fit|right] (icon_drawable.png)


^ Another custom Drawable that we’ve been using internally is an Font Drawable.

^ From a very high level, this drawable type lets us take the SVG text character from a font file and turn it into a drawable.  


^ We can then render those drawables into any colors or sizes that we want and the come out pixel perfect, independent of your screen density.  Our designers are big fans.

—

##IconDrawable
![fit|right] (icon_drawable.png)

![100%|right](icon_documentation.png)



^ A lot of what we’ll talk about today comes from my things I learned while creating this custom drawable type for fonts.


---

# Custom Drawables

^ cool. so how do these work?

---

# Using built-in Drawables

/res
../drawable
  .. layered_drawable.xml

````xml
<layer-list xmlns:android="http://schemas.android.com/apk/res/android">
    <!-- Transparent top line for evenness -->
    <item>
        <shape android:shape="oval">
            <solid android:color="@color/transparent"/>
            <corners android:radius="@dimen/gen_avatar_corners_small"/>
            <padding
                android:top="@dimen/gen_avatar_border_shadow"/>
        </shape>
    </item>
    <!-- Light grey -->
    ...
````

^ So if I was creating a built-in drawables I may do something like this. I’d create an XML file in a drawables folder
and then reference it in a view, set it as a background for a view, etc, like this.

---

# Using built-in Drawables

layered_drawable.xml

````xml
<layer-list xmlns:android="http://schemas.android.com/apk/res/android">
    <!-- Transparent top line for evenness -->
    <item>
        <shape android:shape="oval">
            <solid android:color="@color/transparent"/>
            <corners android:radius="@dimen/gen_avatar_corners_small"/>
            <padding
                android:top="@dimen/gen_avatar_border_shadow"/>
        </shape>
    </item>
    <!-- Light grey -->
    ...
````

layout.xml

```xml
<View 
	...
	android:background="@drawable/layered_drawable"
/>
````

^ So what happens if I make a custom drawable?

---
## Custom Drawable

/res
../drawable
  ..custom_drawable.xml

````xml
<custom-drawable xmlns:android="http://schemas.android.com/apk/res/android"
	xmlns:app="http://schemas.android.com/apk/res-auto"
	app:icon="@id/ic_etsy_e"
	app:color="@color/etsy_orange" >
</custom-drawable>
````

^ let's say I've got a custom drawable. It has a few properties, and I'm setting the color onto it.

----

## Custom Drawable

custom_drawable.xml

````xml
<custom-drawable xmlns:android="http://schemas.android.com/apk/res/android"
	xmlns:app="http://schemas.android.com/apk/res-auto"
	app:icon="@id/ic_etsy_e"
	app:color="@color/etsy_orange" >
</custom-drawable>
````

layout.xml

```xml
<View 
	...
	android:background="@drawable/custom_drawable”
/>
````

^ Just like the previous item, then you set it as the background for you View

^ If you ignore the warnings that Android Studio gives you for this, and just go ahead and build and deploy the project, it will compile and the app will launch.

—

##Runtime Error

````
Caused by: org.xmlpull.v1.XmlPullParserException: 
	Binary XML file line #2: __invalid drawable tag custom__
````

^ Only to crash, with a runtime error that looks something like this.
Ok, so what's going on here?

---

##Drawable.java

````java
	public static Drawable createFromXmlInner(Resources r, XmlPullParser parser, AttributeSet attrs)
	throws XmlPullParserException, IOException {
		Drawable drawable;

		final String name = parser.getName();

		if (name.equals("selector")) {
			drawable = new StateListDrawable();
		} else if (name.equals("level-list")) {
			drawable = new LevelListDrawable();
		} else if (name.equals("layer-list")) {
			drawable = new LayerDrawable();
		} else if (name.equals("transition")) {
			drawable = new TransitionDrawable();
		} else if (name.equals("color")) {
			drawable = new ColorDrawable();
		} else if (name.equals("shape")) {
			drawable = new GradientDrawable();
		}
		...
		} else {
			throw new XmlPullParserException(parser.getPositionDescription() +
			": invalid drawable tag " + name);
		}

		drawable.inflate(r, parser, attrs);
		return drawable;
	}

````

^ It turns out that the custom drawable types that are permitted are hard coded into
the Drawable class.  Anything other than a built-in drawable will crash your app.

---

# Seriously?

---

![75%] (android-deal-with-it.gif)

^ overriding this static call to inject your own drawable type is left as an exercise for the reader

---

# Good ol' Java

````java
    CustomDrawable drawable = new CustomDrawable();
    drawable.setColorFilter(Color.BLACK, PorterDuff.Mode.DST);
    mView.setBackground(drawable);
            
````

^ So any custom drawables that we make will have to be instantiated via their Java class and assigned to the view either through a custom view layout or through set up on view construction.  

^ So it's not as easy to maintain as the XML way but we can get away with it. Plus we want the custom drawing.

^ Ok, let's dive into building custom Drawables.

---

# The Drawable API

---

## Methods to @Override:

- draw(Canvas canvas)
- getOpacity()
- getIntrinsicHeight/Width()
- getMinimumHeight/Width()
- getConstantState() and mutate()

^ so let’s go through a few of the API calls that are key to creating a custom drawable.
This is by no means a comprehensive list.

——

`@Override`
`draw(Canvas canvas)`

---

`@Override`
`draw(Canvas canvas)`

- Just like a View's drawing.
- Call ```canvas.drawSomething(Paint)``` methods.
- Transformations, Rotations, Shaders all still apply.
- Use `getBounds()` to determine drawing area

^ Fairly self explanatory. Just like a normal view. You call the draw methods on the canvas with a Paint object.
^ You can do transformations, rotations, etc on the canvas and use Shaders on Paint.
^ You should use the bounds of the view as set here.

---

`@Override`
`getOpacity()`


---

`@Override`
`getOpacity()`

ColorDrawable.java

````java
    public int getOpacity() {
        switch (mState.mUseColor >>> 24) {
            case 255:
                return PixelFormat.OPAQUE;
            case 0:
                return PixelFormat.TRANSPARENT;
        }
        return PixelFormat.TRANSLUCENT;
    }
````


^ This is best explained by this snippet from Color Drawable.

^If we grab the alpha value out of the color that we’ve set
we return OPAQUE for an alpha value of 255 (1f)
If it's see alpha value of 0, this drawable is transparent
Any other alpha value, you're TRANSLUCENT.

^The other option is PixelFormat.UNKNOWN

---

`@Override`
`getIntrinsicHeight\Width()`


^ The intrinsic height/width of the drawable. 

---

`@Override`
`getIntrinsicHeight\Width()`

ColorDrawable -> Drawable

````java
    public int getIntrinsicHeight() {
        return -1;
    }
````

^ The intrinsic height/width of the drawable is what height of the contents of the drawing in its 'basic' form.
^ In the case of a solid color it has no intrinsic height. In the base of a Bitmap it might be the Bitmap's height or a scaled value.
^ Where there is no intrinsic height you return -1, else the value.

^ this is the call that view queries during the measurement pass

---

`@Override`
`getMinimumHeight\Width()` 

^ This is the minimum height / width of your drawable class. It’s called in 
`getSuggestedMinimumHeight()` in the View class.

---

`@Override`
`getMinimumHeight\Width()` 

Default: returns 0


^ The default implementation of Drawable returns 0 instead of -1. 

^ If you’re writing custom views that rely on a Drawable, you’ll want to call  getIntrinsic, not getMinimum

---

`@Override`
`getConstantState() and mutate()`

^ By default, every drawable inflated from the same resource shares the same state, called the ConstantState

^ When you modify the state of one instance of a drawable, all the other instances inflated from the same resources will get the same modification.

—

`@Override`
`getConstantState()`

![right|65%] (constant_state.png)

—

`@Override`
`mutate()`

- Make this drawable mutable, independent of other instances.
- Basically creates an ‘independent’ state.

![right|65%] (constant-state-mutate.png)

^ mutate allows you to create a new, independent constant state value for an instance of a drawable.

^ This is useful when you want to modify properties of drawables loaded from resources

^ once you’ve made the drawable mutable, calling mutate() will have no effect.


---

# Drawables + Views

^ Ok, so let's figure out how the Drawable's work with Views.

---

# Drawable.Callback

—

## Drawable.Callback

```java
	 public static interface Callback {
	 	
	 	public void invalidateDrawable(Drawable who);
	 	
	 	public void scheduleDrawable(Drawable who, Runnable what, long when);
	 	
	 	public void unscheduleDrawable(Drawable who, Runnable what);
	 	
	 }
```

^ The Drawable.Callback is how the View and Drawable talk. Its a very simple
^ interface that allows the Drawable to tell the View it needs to be redrawn. The View then 
^ invalidates itself in the area the Drawable occupies. Let's focus on the invalidateDrawable method.

---

## Drawable.Callback

````java
	public class View implements Drawable.Callback {
		
		...
		
		public void setBackground(Drawable d) {
			mBackground = d;
			d.setCallback(this);
		}
		
		...
		
		public void invalidateDrawable(Drawable drawable) {
    	    ...
        	    final Rect dirty = drawable.getBounds();
            	final int scrollX = mScrollX;
	            final int scrollY = mScrollY;
    	        invalidate(dirty.left + scrollX, dirty.top + scrollY,
        	            dirty.right + scrollX, dirty.bottom + scrollY);
	        ...
    	}
    	
    	...
    
    }
````

^ Here's abbreviated code from the Android View base class. 
^ When a Drawable is set for something, like the background, the drawable's callback is assigned to be that View.
^ When the Drawable calls back that it needs to invalidate, the Drawable's Bounds are used to invalidate that region of the View on screen.

---

## Drawable.Callback

```java
	public abstract class Drawable {
		private WeakReference<Callback> mCallback = null;
		
		...
		
		public void invalidateSelf() {
    	    final Callback callback = getCallback();
	        if (callback != null) {
            	callback.invalidateDrawable(this);
        	}
    	}
    	
    	...
		
	}
```

^ Here's the Drawable side of things. When it should redraw call invalidateSelf() and the View will redraw you.
^ The single Callback is referenced. If you needed multiple callbacks triggered for one invalidateSelf() call you'll need to use a wrapper.

---

## Drawable Bounds

—

## Drawable.java
`setBounds(Rect rect)`
`setBounds(int left, int top, int right, int bottom)`

^ sets the bounds or drawing coordinates available for the drawable to draw itself in.

—

`setBounds(Rect rect)`
`setBounds(int left, int top, int right, int bottom)`

![right|fit] (view_coordinates.png)

^ Coordinates for drawing a view are measured from the top left corner.  Every View calculates its own coordinates.

---

`setBounds(Rect rect)`
`setBounds(int left, int top, int right, int bottom)`

![right|fit] (drawable_coordinates.png)

^ The drawable’s coordinates are calculated then with reference to the View’s coordinate system.

---

## View Render Passes
- Measurement
- Layout
- Draw

^ Views are kind of like complicated wrappers for drawables. (Very complicated wrappers).
^ The drawable interacts with the View during the measurement, layout, and draw cycles

——


## View Render Passes
- __*Measurement*__
- Layout
- __*Draw*__

^ Views are kind of like complicated wrappers for drawables. (Very complicated wrappers).
^ The drawable interacts with the View during the measurement, layout, and draw cycles

——


### onMeasure
- View queries the Drawable for its desired size `getIntrinsic…()`


—


### onMeasure
- View queries the Drawable for its desired size `getIntrinsic…()`
- View uses the dimens of the Drawable to calculate its dimens and reports back its size to its parent

^ Every view does this a little bit differently

—

### onDraw
- Calculates the bounds for that drawable*

—

### onDraw
- Calculates the bounds for that drawable*
- Calls `setBounds()` on the drawable*

^ * Calculating and setting the bounds for the drawable can happen any time after the measurement of the view

—

### onDraw
- Calculates the bounds for that drawable*
- Calls `setBounds()` on the drawable*
- Calls `draw(canvas)` on the drawable

^ It’s worth mentioning that the contract between the drawable and the view that you’re constructing isn’t set in stone.  How they interact depends on the view that you’re drawing.  

—

### A Few Notes:
Views assume that the drawable knows what size it wants to be before the bounds get set.

^ You may need to know the size of the drawable before knowing how much space you have to draw in.

—

### A Few Notes:
Views assume that the drawable knows what size it wants to be before the bounds get set.

Every View computes the drawable’s bounds differently.

^ Next every view computes its bounds differently

—

### View Bound Calculations:

ColorDrawable.java

````java
getIntrinsicHeight() { returns -1; }
```

^ so let’s consider the what happens when a drawable passes back a height and width of -1, which means it has no set dimensions.

—

### View Bound Calculations:

ColorDrawable.java

````java
getIntrinsicHeight() { returns -1; }
````

CompoundButton
`setBounds(-1, -1, -1, -1);`

^ The compound button will return -1.

—

### View Bound Calculations:

ColorDrawable.java

````java
getIntrinsicHeight() { returns -1; }
````

CompoundButton
`setBounds(-1, -1, -1, -1);`

ImageView 
`setBounds(leftMax, rightMax, topMax, bottomMax)`

^ For the drawable in an ImageView, if you pass back dimensions of -1, the imageView will set the bounds to be the total calculated size of the view.

—

# The Future of Drawable 

^ Ok, so we’ve got some time left. Let's talk about some new drawable types coming up in L.

---

# Ripple

---

![75%|loop] (ripple_example.mov)


^ The Ripple is a fundamental part of Material Design presented at I/O. It provides feedback
^ when the user interacts with almost any UI element. There's a few different interesting parts to it.
^ First you'll notice the Ripple originates and moves with the TouchEvents going on.

---


![35%|loop] (ripple-masked.mov)

^ Second you'll notice the Ripple can be masked to another Drawable, such as a Bitmap.


---

# Ripple

```
	public class Drawable {
		...
	 	
    	public void setHotspot(float x, float y) { /* compiled code */ }
	 	
	 	...
	}	
```

^ The location of the TouchEvent is called the Hotspot. 
^ In L they added some new methods to Drawable to update the location of the Hotspot.
^ It's pretty clear that somewhere in a View class the TouchEvent locations are being passed to the Ripple.

---

# Pre-L?


^ So Drawable has been fundamentally altered in L. But let's try and bring this to our Pre-L apps. 
^ Drawing the ripples is fairly straight-forward. You're drawing two circles.
^ One covers the background and the other follows the TouchEvents. 
^ We won't dive into how to draw the circles because once L is open sourced we can get the animation timings and effects exact. 
^ For now let's look at the two major features which is the Hotspot and the Mask.

---

# RippleDrawableCompat

```java
	public class RippleDrawableCompat extends LayerDrawable {	
	
	}
```

^ Ok, let's start building a Drawable that mirrors the RippleDrawable in L
^ Checking out the compiled code in L we see that RippleDrawable extends LayerDrawable so let's do that

---

# RippleDrawableCompat

```java
	public class RippleDrawableCompat extends LayerDrawable {
	
		private Drawable mask;
	
		public RippleDrawableCompat(Drawable content, Drawable mask) {
			super(content != null ? new Drawable[] { content } : new Drawable[] { } );
			this.mask = mask;
		}	
				
	}
```

^ Next, let's match the constructor arguments
^ The LayerDrawable super class expects an array of Drawable and can't be null.
^ Here we check if the content is null and create an empty array.
^ The final argument is a Drawable to use as a mask. For now let's just keep a reference to it in our class.

---

# RippleDrawableCompat

```java
	public class RippleDrawableCompat extends LayerDrawable {
	
		private ColorStateList colors;
		
		public void setColor(ColorStateList colors) {
			this.colors = colors;
		}
				
	}
```

^ The Color of the Ripple is provided as a ColorStateList
^ The ColorStateList will provide a color based on the current state, such as Pressed

---

# RippleDrawableCompat

```java
	public class RippleDrawableCompat extends LayerDrawable {
	
		private float hotspotX;
		private float hotspotY;
	
		public void setHotspot(float x, float y) {
			this.hotspotX = x;
			this.hotspotY = y;						
		}
	
	}
```

^ Now, let's keep track of the Hotspot location 

---

# RippleDrawableCompat

```java
	public class RippledImageView extends ImageView {
		
		...
		
		@Override
    	public boolean onTouchEvent(MotionEvent event) {
        	updateHotspot(event.getX(), event.getY());
        	return super.onTouchEvent(event);
    	}
    	
    	private void updateHotspot(float x, float y) {
    		
    		Drawable background = getBackground();    		 
        	if (background != null && background instanceof RippleDrawableCompat) {
            	((RippleDrawableCompat) background).setHotspot(x, y);
	        }

    	    Drawable src = getDrawable();
        	if (src != null && src instanceof RippleDrawableCompat) {
            	((RippleDrawableCompat) src).setHotspot(x, y);
	        }
	        
    	}
		
	}
```


^ To add support for Hotspot we need to extend any View who's using the RippleDrawableCompat to pass its TouchEvents along.
^ We do this by checking any set Drawables for their class type and then calling the setHotspot() method
^ This means you'll be using custom Views through your app but chances are you're doing this already.

---

# Mask

![right|125%] (mask-example.png)

^ Next, we want to support the Drawable Mask
^ Here you can see the effect. The Ripple is limited to draw only in the area of the Drawable
^ In this case we're using the sample Android bitmap icon as the mask.
^ From experience we know that BitmapShader's are an easy way to do masking.
^ To use the BitmapShader first we need to convert our Drawable into a Bitmap.

---

# RippleDrawableCompat

```java
	
	private Bitmap convertDrawableToBitmap(Drawable mask, Bounds bounds) {
		
		Bitmap maskBitmap = Bitmap.createBitmap(bounds.width(), bounds.height(), Bitmap.Config.ALPHA_8);
		
	}	
```

^ First, we create a bitmap thats the correct size, the width and height will come from our Drawable bounds
^ We use the Bitmap Config Alpha 8 which only stores a single channel for translucency.
^ Essentially where there are pixels there is a value otherwise its transparent.

--- 
# RippleDrawableCompat

```java
	
	private Bitmap convertDrawableToBitmap(Drawable mask, Bounds bounds) {
		
		Bitmap maskBitmap = Bitmap.createBitmap(bounds.width(), bounds.height(), Bitmap.Config.ALPHA_8);

	    Canvas maskCanvas = new Canvas(maskBitmap);
	}	
```

^ Next we create a Canvas using the Bitmap

--- 
# RippleDrawableCompat

```java
	
	private Bitmap convertDrawableToBitmap(Drawable mask, Bounds bounds) {
		
		Bitmap maskBitmap = Bitmap.createBitmap(bounds.width(), bounds.height(), Bitmap.Config.ALPHA_8);

	    Canvas maskCanvas = new Canvas(maskBitmap);

        mask.setBounds(bounds);		
        mask.draw(maskCanvas);
        
        return maskBitmap;
	}	
```

^ Finally we set the bounds on our Drawable mask and have it draw into the Canvas
^ Then we just return our Bitmap

---

# RippleDrawableCompat

```java
	public class RippleDrawableCompat extends LayerDrawable  {	
		
		@Override
    	protected void onBoundsChange(Rect bounds) {
        	super.onBoundsChange(bounds);
        	        	
        	// we have a Drawable to use as a mask
        	if (mask != null) {

            	Bitmap maskBitmap = convertDrawableToBitmap(mask, bounds)

	            // this shader will limit where drawing takes place to only the mask area
    	        maskShader = new BitmapShader(maskBitmap, Shader.TileMode.CLAMP, Shader.TileMode.CLAMP);

        	}
                    
        }
    }
```

^ We know our size when onBoundsChange is called. 
^ Now we just call our method to convert the Drawable into a Bitmap and use that in our BitmapShader

---
# BitmapShader

![right|125%] (mask-example-no-color.png)

^ Here's the effect we get. The mask works but the color isn't what we want. 
^ Its only using the value which is stored when there's no transparency from the Bitmap, which is gray.
^ Next let's get the color of the Ripple and color our masked area when drawing.

---

# RippleDrawableCompat

```java
	public class RippleDrawableCompat extends LayerDrawable {	
			
		@Override
	    protected boolean onStateChange(int[] state) {
	    	
	    	// get the color for the current state                
	    	int color = colorStateList.getColorForState(state, Color.TRANSPARENT);	    
	    	
	    }
	    
	}
```

^ When our state changes we can now get the color of our ripple for this state.

---

# RippleDrawableCompat

```java
	public class RippleDrawableCompat extends LayerDrawable {	
			
		@Override
	    protected boolean onStateChange(int[] state) {
	    	
	    	// get the color for the current state                
	    	int color = colorStateList.getColorForState(state, Color.TRANSPARENT);
	    			    			    
		    LinearGradient gradient = new LinearGradient(0, 0, 0, 0, color, color, Shader.TileMode.CLAMP);
    	    ComposedShader shader = new ComposeShader(maskShader, gradient, PorterDuff.Mode.SRC_IN);	                	                
                    
            paint.setShader(shader);
          	    	
	    }
	    
	}
```

^ The ComposeShader allows us to combine multiple shaders together using a PorterDuff mode.
^ To achieve the affect we want we need to compose our Mask's BitmapShader with a solid color shader using PorterDuff Mode SRC IN
^ Since there is no Solid Color Shader, we use a LinearGradient with the same color for start and end values
^ Once composed this will limit drawing to the masked area and draw the correct color.

---

![35%|loop] (ripple-compat-demo.mov)

^ Here's a demo of the effects running on a Jelly Bean Emulator.

---

# Thank You!

^ If you’ve got questions or are interested in what we’re up to at Etsy, come say hi at our booth

---


# __*getBounds()*__


# [fit] The story of Drawables and their View masters

#### __*Jamie Huson*__ + __*Lisa Neigut*__ | __20 Sept 2014__


