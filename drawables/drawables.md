
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

# Lollipop

^ Ok, let's talk about changes to Drawables in Lollipop. 

---

# Ripple

---

![75%|loop] (ripple_example.mov)


^ Material Design's Ripple was present in the L Preview which gave us a hint about things to come. 
^ We noticed that the Drawable reacts to touch input.

---


![35%|loop] (ripple-masked.mov)

^ Second you'll notice the Ripple can be masked to another Drawable, such as a Bitmap.
^ The design constraints of Material Design, including Ripples, seems to be the driving force to the API changes.

---

# Lollipop - Touch Events

```
	public class Drawable {
		...
	 	
    	 public void setHotspot(float x, float y) {}   

    	 public void setHotspotBounds(int, int, int, int) {}
	 	
	 	...
	}	
```

^ The ability to track TouchEvents is new for Drawables.
^ The location of the TouchEvent is called the Hotspot. 
^ To update the hotspot simply call the new method setHotspot() with the coordinates.
^ You can also update the bounds of the hotspot independently of the Drawable's general bounds
^ TODO: Where does a View call this?

---

# Lollipop - Outline

```
	public class Drawable {
		...
	 	
    	 public void getOutline(Outline outline) {}   
	 	
	 	...
	}	
```

^ New in Lollipop is the ability to set elevation.
^ Elevation is a key part of Material Design that allows different planes of content to be distinguishable from one another.
^ Android draws shadows for Views based on their elevation.
^ The shape of those shadows is defined by a View's Outline.
^ Since Drawables are used as the background of Views the Drawable itself provides an Outline through a new getOutline() method.
^ By default the Drawable's outline is simply its bounds but this can be modified based on the Drawable content.

---

# Lollipop - Dirty Region

```
	public class Drawable {
		...
	 	
    	 public Rect getDirtyBounds() {
    	 	return getBounds();
    	 }   
	 	
	 	...
	}	
```

^ The dirty region is the area that will be invalidated and redrawn.
^ Pre-Lollipop invalidation would cause the entire Bounds of a Drawable to be re-drawn.
^ This is no longer the case in Lollipop. By default the bounds are used, but this can be modified by returning a different Rect to invalidate only a region within the Drawable.

---


![35%|loop] (ripple-bounds-dialer.mov)


^ Ripples use this to define the dirty region to extend into the hosting View's parent area when its unbounded such as in Dialer.

---

# Lollipop - Theme


```
	public class Drawable {
		...
	 	
    	 public void applyTheme(Theme) { }

    	 public boolean canApplyTheme() { }

    	 public Drawable createFromXml(Resources, XmlPullParser, Theme)

    	 public Drawable createFromXmlInner(Resources, XmlPullParser, AttributeSet, Theme)

    	 public void inflate(Resources, XmlPullParser, AttributeSet, Theme)
	 	...
	}	
```

^ In Lollipop the application Theme now supplies the color palette of the app through attributes.
^ This is supported in AppCompat as well.
^ Ripples are using this information to create a complementary ripples color that matches the app theme.
^ You can see here that the methods for inflation and creation  of the Drawable now includes the Theme of the app.

---


# Lollipop - Tint


```
	public class Drawable {
		...
	 	
    	 public void setTint(int)

    	 public void setTintList(ColorStateList list)

    	 public void setTintMode(Mode)

	 	...
	}	
```

^ And to automatically apply the color there's a series of methods to apply the Tint color to the drawable.
^ It's important to note that it's up to the Drawable sub-class to use that color in some way.
^ The Mode in setTintMode refers to is the PorterDuff Mode which determines how the color should be applied to the drawable's content.
^ Pro-Tip: Setting a ColorFilter will override the Tint supplied either here or through the Theme

---


# Lollipop - ConstantState

```java

public class ConstantState {
	

	public boolean canApplyTheme()

	public Drawable newDrawable(Resources, Theme)

}

```

^ The ConstantState class also got a slight update to support theme attributes. 
^ In addition the newDrawable() method now takes in a Theme to create a clone with Theme attributes intact.

---

# VectorDrawable

^ It's also worth mentioning the VectorDrawable and its sibling AnimatedVectorDrawable were added in Lollipop.
^ This allows you to load Vector graphics from SVG and display them as a Drawable.
^ The main parts of this are the SVG loader. 

---

# Backwards Compatibility

MrVector https://github.com/telly/MrVector

^ Since the VectorDrawable's design constraints didn't require big changes to the API it's actually been backported already.
^ Recent hidden changes to the Support Library also suggest Google is working on official support for pre-Lollipop devices also.
^ Ok, what about Ripples? We hinted at efforts to do this at DroidCon NYC.

---
 

![center|fit] (sad-kitty.jpg)


^ Ripple's implementation heavily relies on the recent Lollipop API changes, making it not straight forward.
^ Let's talk about those challenges.

---

# Ripple

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

^ Since the Ripple follows the finger, it needs to know where the TouchEvents are occuring.
^ Pre-Lollipop View's don't pass this information to Drawables.
^ In order to implement this you'll need to create a custom View subclass and pass that info yourself.
^ This means that any View using a Ripple will need to be custom. 
^ This also means that integration with libraries like AppCompat become more difficult since they don't handle custom Views well.

---

# Ripple

```java
@Override
public void draw(@NonNull Canvas canvas) {
        // Clip to the dirty bounds, which will be the drawable bounds if we
        // have a mask or content and the ripple bounds if we're projecting.
        final Rect bounds = getDirtyBounds();
        final int saveCount = canvas.save(Canvas.CLIP_SAVE_FLAG);
        canvas.clipRect(bounds);
        
        drawContent(canvas);
        drawBackgroundAndRipples(canvas);
        
        canvas.restoreToCount(saveCount);
}
```

^ Implementation wise if you peek at RippleDrawable you'll see the Drawable delegates the logic of Ripples to a Ripple class.
^ The Drawable simply keeps a list of Ripples and calls through to them in draw() 

---

# Ripple

```java
public boolean draw(Canvas c, Paint p) {
        final boolean canUseHardware = c.isHardwareAccelerated();
        if (mCanUseHardware != canUseHardware && mCanUseHardware) {
            // We've switched from hardware to non-hardware mode. Panic.
            cancelHardwareAnimations(true);
        }

        mCanUseHardware = canUseHardware;
        final boolean hasContent;
        if (canUseHardware && (mHardwareAnimating || mHasPendingHardwareExit)) {
            hasContent = drawHardware((HardwareCanvas) c, p);
        } else {
            hasContent = drawSoftware(c, p);
        }

        return hasContent;
}
```

^ We can see in the Ripple's draw it's choosing to draw using Hardware acceleration if available and fallback to Software.
^ Given ripples are used all over on potentially every View on screen this could quickly cause performance hits if using Software alone.
^ Ripples are constantly animating. To do this efficiently there's a hidden RenderNodeAnimator class.
^ These hidden RenderNode classes interact with the native OpenGL canvases and keep their own DisplayLists to get higher performance.

---

# Ripple

Search: 

Android Graphics Pipeline Button to FrameBuffer"

AOSP:

https://source.android.com/devices/graphics/index.html


^ DisplayLists were used as part of Project Butter to speed things up by re-ordering drawing calls to be more effecient for the GPU.
^ If you want to understand more about how this works and why it acheives better performance there's a great series of blogposts. Google it
^ You can also read the Rendering and Graphics pipeline documentation on the AOSP site.

---

![center|fit] (res-drawable-folders.png)

^ It may be possible to backport this by extracting out all the classes but on older devices (especially pre Project Butter) the performance gains may not be possible to achieve for application use.
^ For most developers sticking with Selectors on pre-Lollipop devices is straight-forward and supported using resource qualifiers. 

---

# Thank You!

^ If you’ve got questions or are interested in what we’re up to at Etsy, come say hi at our booth

---


# __*getBounds()*__


# [fit] The story of Drawables and their View masters

#### __*Jamie Huson*__ + __*Lisa Neigut*__ | __20 Sept 2014__


