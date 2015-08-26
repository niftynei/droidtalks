
# Android Development 
## A Pirate's __CODE__ 

---

# Android Development 
## A Pirate's __Guidelines__

--- 

#[fit] Where did these come from?

^ so i started at Electric Objects in January
^ i started where, i'm sure most of us end up every once in a while:
^ coming in to take over a project from some consultants
^ which is to say more or less starting over from scratch
^ (sorry StartUp Giraffe)
^ when i got there, i found that myself asking this question:

---

# HOW DO I START AN ANDROID APP?

^ um.  
^ # awkward pause #
^ good question.
^ to be honest, i kind of had no idea.

--- 

![100%](wireframe_set.png)

^ if you're a lucky android dev, you probably start off with something that looks like this.
^ mock ups that your designer made.
^ if you're not lucky, you'll definitely start out with something that looks like this

---

![100%](ios_wireframe_set.png)

^ mock ups for an iOS app.

---

^ Ok, I've got some designs, so clearly we're not at square one.
^ so let's refine the question a little bit.

---

# HOW DO I MAP THESE MOCK UPS TO ANDROID 'PIECES'?

---

^ my only other experience with this has been at Etsy
^ when i started at Etsy, there were only two of us on the Android team
^ myself, who had never worked as a software developer before
^ much less written an entire android app.
^ and our Senior Android Dev Tim, 
^ who was actually an iOS dev who decided Android was the next big thing
^ and that Etsy needed to get with it.

![left](lisa_droid.png)
![right](tim_droid.png)

---

^ now tim was probably one of the smartest app dev's i've ever known
^ he'd done a bunch of reading on Android apps and was really into
^ this new app dev paradigm called 'Fragments'
^ the original Etsy Android app looked something like this

~~ image of Android app with Fragments ~~

---

^ there's nothing wrong with this, really.
^ it seemed to work ok, but at some point i think we started running into some problems
^ the app grew in size, we did a big redeisgn, one that added tablet layouts
^ by the time I left Etsy, when you were adding a new feature to the app
^ this is more or less what our app looked like 

~~ image of Android app with 1:1 Fragment to Activity ratio ~~

^ hmmmm. # pause #
^ when i got to EO, i decided to improve upon this architecture just a bit
^ nothing too crazy or revolutionary

---

^ and in the process created my first "Android App Dev Guideline"

# Android App Dev Guideline #1

---

# Android App Dev Guideline #1
## There is a one to one correspondence between a wireframe screen and an Activity

~~ image of mapping of 1 Activity to 1 mockup ~~

^ what's nice about this?

---

# Super easy to get started writing a new application

^ done, no more thinking. 
^ once the designer is done, i've got my blueprint, now just to build the house.

---

![left|fit](star_trek_blueprints.jpg)
![right|fit](wireframe_set.png)

^ okay, so you're ready to get started making your app.

---

^ so as you do, i start looking for similarities between these mock ups
^ hmm, these look very similar

![left|fit](okay_activity.png)
![right|fit](success_activity.png)

---

^ so you start building something that looks like this

```java
OkayActivity.java
setContentView(R.layout.activity_generic.xml);
```

```java
SuccessActivity.java
setContentView(R.layout.activity_generic.xml);
```
^ so far so good. 
^ the success message is just an image anyways, screw i18n
^ both of them have buttons. 
^ less code to maintain, whoo hoo. looking good.

--- 

^ then your designer is looking at your app a few days later
^ and you have a conversation that goes a little something like this

---

` luke: hey lisa! 👋`

---

`luke: hey lisa! 👋`
`lisa: hey! 👋🏽`

---

`luke: the app's looking really great! 👍🏻`

---

`luke: the app's looking really great! 👍🏻`
`lisa: /me blushes ☺️`

---

`luke: just one thing`

^ # pause to let people read it #
^ # say # stomach starts to drop a bit

---

`luke: just one thing`
`luke: the user needs a way to logout from the success screen`

---

![100%](success_activity.png)
![100%](success_activity_logout.png)

---

`lisa: /me thinks furiously`

---

`lisa: /me prepares diatribe on the Android Back Button`

---

`lisa: /me realizes she is vastly outnumber by iOS devices`

---

^ # look left.  look right. (furtively) #

---

`lisa: /me ctrl-C; ctrl-V`

---

```java
OkayActivity.java
setContentView(R.layout.activity_generic.xml);
```

```java
SuccessActivity.java
setContentView(R.layout.activity_slightly_less_generic.xml);
```

---

^ and that's how i came up with my second guideline

# Android App Dev Guideline #2

---

# Android App Dev Guideline #2
## Every Activity has its own layout file

^ which, in practice, looks like this:

---

^ i'm sure some of you are thinking, lisa you're daft.
^ which is totally true.
^ what about this other, more general programming best practices principle

---

# PRGRAMMING BEST PRACTICES PRINCIPAL 101

---

## DRY: Don't Repeat Yourself

^ one i'd say that's hogwash.
^ two, i'd say that's it's more like a guideline than a hard and fast rule.
^ and that if you've got view code that you're repeating
^ that's where my next guideline comes in

---

^ now, there's one exception to this rule in the Electric Objects app
^ that I wish i had followed my own advice on.
^ that's the LoginActivity.java

---

![left|100%](login_activity.png)
![right|100%](sign_up_activity.png)

---

![left|100%](login_activity_marked_1.png)
![right|100%](sign_up_activity_marked_1.png)

---

![left|100%](login_activity_marked_2.png)
![right|100%](sign_up_activity_marked_2.png)

---

![left|100%](login_activity_marked_3.png)
![right|100%](sign_up_activity_marked_3.png)

^ these screens ended up having a lot of differences which would have been far easier to deal
^ with as layout code rather than logic statements.
^ there was one redeeming quality to this screen, and that was my next guideline
^ the majority of the branching logic for views on this screen is wrapped up in a single view class

---

![fit](LoginView_java.png)

^ which, as it happens, is guideline number 3

---

# Android App Dev Guideline #3

---

# Android App Dev Guideline #3
## Reusable View code belongs in a View class

^ if you've got view code that you want to use again, put it in a view class.

---

![100%](error_view.png)

^ here's an example of a view class that is in practically every every
^ activity layout of our app

^ there's a corollary to this rule.  sometimes you can get away without writing a view class
^ for repeated view code

---

# Android App Dev Guideline #3A
## Reusable View attributes belong in a style

^ you can get away with some pretty cool, flexible view code with some creative styling.

---

![left|fit](bluetooth_portrait.png)
![right|fit](wifi_portrait.png)

^ for example, here's a view that gets repeated a few times.

---

![left|fit](bluetooth_landscape.png)
![right|fit](wifi_landscape.png)

^ that i wanted to resize nicely for different views.

---

![left|fit](image_view_style_portrait.png)
![right|fit](image_view_style_land.png)

^ I ended up using different styles for the landscape and portrait layouts
^ but that defined as the same style, that i can then apply to each of these
^ generic ImageViews

---

![100%](image_view_large.png)

^ sweet. now i've got less code to worry with and it'll be easy to change a style if this every changes.
^ so that just about wraps up any code that you may need or want for views.

---

^ you may have noticed that i haven't mentioned fragments yet.
^ that's because i don't really need them.
^ i try to make it a point not to need them.
^ which is android app dev guideline number 4

---

# Android App Dev Guideline #4

---

# Android App Dev Guideline #4
## Fragments are not simple.  Don't use fragments.*

---

# Android App Dev Guideline #4
## Fragments are not simple.  Don't use fragments.*

#### *unless it is absolutely necessary

^ so it's been a while since i've used a fragment for view code.
^ and i was having trouble remembering why this was such good advice.
^ then i was talking to a friend of mine, kasra who works at stack exchange.
^ i'd like to share with you part of our conversation

---

![150%](kasra_convo_2.png)

---

![150%](kasra_convo_3.png)

---

^ ![150%](kasra_convo_4.png)

^ ---

^ raise your hand if you've ever had a conversation
^ or run into a similar bug with fragments before?

^ # pause to let people raise their hands #
^ now look around you.  let's pause for moment, a sad moment,
^ to just feel sad about fragments.

---

#💧🌹🌹🌹🌹

^ # moment of sadness for how sad fragments are #

^ that's all i'm going to say about using fragments for view code.

---


# Android App Dev Guideline #5

^ so the next guideline
^ actually the next several guidelines
^ really come from a debates we used to have at Etsy
^ about the best way to handle rotation in an app

---

^ at etsy, it was not uncommon to see something like this in your activty code
^ it's possible they've moved away from this these days, but also possible not

```java
@Override
protected void onConfigurationChanged(Configuration config) {
 ...
 \\ usually something way hacky
 ...
}
```

---

^ when we did this we ended up with a lot of weird bugs.

- languages not changing correctly
- views not resizing properly
- strange invocations to .onSaveInstanceState() in `onConfigurationChanged`

---

^ i sum these problems up this way:

- BUGGY
- HARD TO TEST
- HARD TO FIX

^ i don't think most people do this, but i don't think it's a bad guideline to have either way.

---

# Android App Dev Guideline #5

## Never override configuration changed.*

^ so that's why it's number 5

---

# Android App Dev Guideline #5

## Never override configuration changed.*
#### * for view code

^ that being said, i'm sure that there's some rational reason for overriding
^ on config changed for non view or state related things. this guideline doesn't
^ apply to that.

---

# Android App Dev Guideline #6

^ the next guideline is closely related to never override configuration changed
^ and honestly, has more to do with fragments than anything

---

# Android App Dev Guideline #6

## Retain nothing but state (and hardware adapters)

^ basically if you're not using fragments for view code
^ and since if you're following guideline FOUR, and not using fragments
^ this guideline probably doesn't much apply to you.
^ my biggest problem with this one had to do with testability

---

- How well do you really know your Android Activity Lifecycle?

^ as soon as you turn this one on, i think you'll find yourself questioning how well
^ you really know your activity lifecycle. the answer is probably not as well as you think.

---

- HARD TO TEST

^ retained fragments are hard to test;
^ and since they're hard to test, they're hard to find bugs with, which means they're also hard to fix.

---

- HARD TO TEST
- HARD TO FIX

---

# Android App Dev Guideline #7

^ so the last guideline
^ is really a reflection of all the hardware adapters that i've had to deal with.
^ and the shift in mentality that i had to make from doing web development

--- 

![fit](web_inspector.png)

^ when you're developing for the web
^ depending on what you're doing of course, 
^ you don't really have to worry too terribly much about maintaining state

^ on android, state is the thing that bites you in the ass every time you sit down.

---

![left|100%](portrait.png)
![right|100%](landscape.png)

^ or turn over.

---

# Android App Dev Guideline #7

## Embrace state machines

^ # read guideline out loud #
^ so you may be thinking to yourself, what is a state machine

---

![100%](simple_state_machine.png)

^ basically, a state machine is an object
^ where the functions available to be preformed are relative to what state
^ the object finds itself in, and where the states
^ are generally ordered, such that the next state you can move to is determined by
^ the current state you find yourself in.

---

^ or in other words...

# A State Machine
- States are explicitly defined
- Can only do certain things in each state
- Where you can go next depends on where you are now

---

^ these come in handy when using hardware adapters, which sometimes you put into fragments
^ that can then be retained.

---

![100%](bluetooth_state_machine.png)

---

![100%](bluetooth_state_machine_in_action.png)

---

- There is a one to one correspondence between a wireframe screen and an Activity
- Every Activity has its own layout file
- Reusable View code belongs in a View class
- Fragments are not simple.  Don't use fragments.*
- Never override configuration changed.*
- Retain nothing but state (and hardware adapters)
- Embrace state machines

---


# Lisa Neigut
##[fit] work @electricobjects
##[fit] me on the internet, @niftynei

![right|100%](eo_logo.png)

---

## ~thank you~
![right|100%](eo_logo.png)

---
