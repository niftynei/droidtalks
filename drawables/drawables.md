
# getBounds()

# [fit] The story of Drawables and their View masters

---

[[ INSERT PIC OF JAIME ]]
[[ INSERT PIC OF LISA ]]

^ Hi, we're lisa and Jaime, we work at Etsy on the Android team.

^ At Devoxx 2013, Cyril Mottier gave a great talk on Mastering Android Drawables. This talk aims to build on what Cyril laid out and go a bit more in-depth into the APIs / how to actually implement a custom drawable, and how they relate to a View’s measurement/layout/draw cycles.

---

# What is a Drawable?

---

[[ INSERT IMAGE OF Cyril Mottier's Definition ]]

^ Borrowing Cyril Mottier’s definition: A drawable is a positioned entity that is to be drawn on a canvas.
Here's a few examples of uses for drawables.

----

NinePatchDrawable

![150%] (nine_patch.png)

^ A drawable can be a 9 patch.

---

ShapeDrawable

````xml
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="oval">
    <solid android:color="@color/blue" />
    <size android:height="@dimen/padding_medium" android:width="@dimen/padding_medium" />
</shape>
````

^ It can be a shape that's specified in XML.

---

StateListDrawable 

** GIF OF A CLICK BUTTON STATE **

^ Drawables are flexible and useful.  You can nest different drawbles into each other to make buttons that press

---

BitmapDrawable

![right] ** IMG OF REPEATING BACKGROUND **

^ Or to repeat a pattern on a background

---

LayerDrawable

** IMAGE OF STACKED DRAWABLES ** 


^ Stack them on top of each other

---

RippleDrawable

** IMAGE OF RIPPLE GIF

^ there's even a new drawable type coming out with L - The Ripple Drawable, that has some nice animations to highlight touch effects

---

** TODO : Show as a hierarchy tree **

^ These are all native drawables, which means you can create them in XML files and then assign them to views also in XML.

---

What Makes Drawables Awesome:


---

What Makes Drawables Awesome:
- Total Separation of View Logic from Code

^ Drawables let you do a lot of setting up button states and how your views will look in 
XML, not in code.

---

# Custom Drawables

---

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

^ So native drawables you can instantiate via code like this. You save it in a drawables folder
and then you can reference it in a view, set it as a background for a view, etc, like this.

---

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

lisas_custom_drawable.xml

````xml
<custom-drawable xmlns:android="http://schemas.android.com/apk/res/android"
	xmlns:app="http://schemas.android.com/apk/res-auto"
	app:icon="@id/ic_etsy_e"
	app:color="@color/etsy_orange" >
</custom-drawable>
````

^ let's say I've got a custom drawable. It has a few properties, and I'm setting the color onto it.
This will actually compile and everything looks fine, other than the hideous red underline that 
Android Studio puts on your drawable folder.  But it will build and launch.  

----

Runtime Error

````
Caused by: org.xmlpull.v1.XmlPullParserException: Binary XML file line #2: invalid drawable tag custom
````

^ Only to crash, with a runtime error that looks something like this.
Ok, so what's going on here?

---

Drawable.java

````
/**
* Create from inside an XML document. Called on a parser positioned at
* a tag in an XML document, tries to create a Drawable from that tag.
* Returns null if the tag is not a valid drawable.
*/
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
the Drawable class.

----

````java
    ShapeDrawable drawable = new ShapeDrawable(new OvalShape());
    drawable.setColorFilter(Color.BLACK, PorterDuff.Mode.DST);
    mView.setBackground(drawable);
            
````

^ So any custom drawables that we make will have to be instantiated via Java code and 
assigned to the view either through a custom view layout or through set up 
on view construction.  

^ Let's go back for a second to what makes drawable code so awesome.

---

What Makes Drawables Awesome:
- Total Separation of View Logic from Code

^ Making custom drawables, in a way, defeats this great separation of concerns that we already established
as one of the main reasons to use them.  So why do we use custom drawables?

---

![] (badge_drawable.png)

^ Because they give you flexibility to do some awesome things.
This is Jesse Hendrickson's Badge Drawable.

---

![left] (icon_drawables.png)
![right](icon_documentation.png)

^ Internally, we've been using them to turn fonts into icons.  Admittedly, it makes our code a bit 
more verbose at times, but it saves time in other ways.

---

# The Drawable API

---

Methods that need to be overridden:

- draw(Canvas canvas)

^ Of course, you need to draw on the canvas for a drawable

---

Methods that need to be overridden:

- draw(Canvas canvas)
- getOpacity()


---

Methods that need to be overridden:

- draw(Canvas canvas)
- getOpacity()
- getIntrinsicHeight/Width()

---

Methods that need to be overridden:

- draw(Canvas canvas)
- getOpacity()
- getIntrinsicHeight/Width()
- getMinimumHeight/Width()

---

Methods that need to be overridden:

- draw(Canvas canvas)
- getOpacity()
- getIntrinsicHeight/Width()
- getMinimumHeight/Width()
- setBounds(Rect bounds)

---

Methods that need to be overridden:

- draw(Canvas canvas)
- getOpacity()
- getIntrinsicHeight/Width()
- getMinimumHeight/Width()
- getConstantState() and mutate()

---

`draw(Canvas canvas)`


^ Fairly self explanatory

---

`getOpacity()`


---

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
OPAQUE is anything without alpha
If it's see through, pass back transparent
if it's a combination, you're TRANSLUCENT.

---

`getIntrinsicHeight`



^ The height of the drawable. 

---

`getIntrinsicHeight`

ColorDrawable

````java
    /**
     * Return the intrinsic height of the underlying drawable object. Returns
     * -1 if it has no intrinsic height, such as with a solid color.
     */
    public int getIntrinsicHeight() {
        return -1;
    }
````

^ TODO: explain. The height of the drawable. 

---

`getMinimumHeight` 

^ Like intrinsic, except minimum that can be passed back is 0, instead of -1 for intrinsic.

---

`getConstantState() and mutate()`


^ TODO: Explain this

---

# Using Drawables with Views

---

# The Drawable Callback

^ todo: jamie?

---

# Drawable Bounds

^ Todo: lisa

---

# The Future of Drawable 

^ Todo: (Go over Ripple Compat here?)

---

New Methods Added to RippleCompat

- setHotspot()
- setHotspotBounds()

^ todo!!

---