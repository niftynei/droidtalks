
A Pirate's Guidelines
~~ to ~~
Android Dev

^ hey, i'm lisa, i run Android development at Electric Objects

---
^ my goal today is to enter into the conversation of android development practices
~~ GOALS: Share some stuff I've Learned ~~
~~ GOALS: Start a conversation about Android dev practices ~~

^ some of the things that i've learned over the past four years of doing android development
^ or really just programming in practice.

^ before i  get started, let's talk a bit about the motivation for this talk.
^ i started doing android back when gingerbread was the newest thing, and i spent the last 2 and a half years
^ learning a lot about programming at Etsy, as one of the first members of their Android team. RIP

^ as an app developer, i'm  mostly self-taught. my interest in Android started
^ mostly as a fascination about what you could *make* with Android code than about the code in and of itself

^ i quickly learned, as i think all of us who stick with programming do, 
^ that it's great if your app works sometimes

~~ gif of an error screen ~~
^ but a crashing app is pretty useless.

^ when i started working at Etsy, i didn't know very much Android.
^ neither did anyone at Etsy.
^ our senior dev on the Android team was an iOS dev
^ who had read all about fragments and decided that our app was going to be 
^ a very small number of Activites, with fragments layered on top of each of them

^ here's some code from those early days:
~~ weird if statements ~~

^ my first big 'ANDROID FEATURE' was writing the cart.  
~~ show picture of the cart ~~

^ to go back to the no crashing thing,  
^ when you're a shopping app, the cart's definitely one of the more important parts of the user experience.

^ i hadn't quite grasped the concept of writing callback clases yet, so we ended up with a file that was more or less 2000 lines long
^ lololol

^ i'd like to take this moment to offer a formal apology to whoever has since inherited this code.
`
~~ I'm So So So Sorry. ~~
~~ *kermit saying sorry?* ~~

-- 

^ a little while later, etsy hired our first real 'senior android dev'
^ we've never talked about it, but i'm pretty sure he cried on his first day

--

^ when i started at Electric Objects a few months ago, 
^ i got my first mock ups from our designer of what our app needed to look like
~~ show first app sketch mock ups ~~`

^ here was an opportunity to do things differently.

^ i thought long and hard about what had been so painful about our last app, 
^ and came up with this short list of 'guiding app principles' 
^ which i'd like share with you all today.

^ before i do though, i'd like to take a moment to talk about the inspiration for these guidelines
~~ image of johnny depp as jack sparrow ~~
^ those of you who remember pirates of the carribbean, may remember this character.
^ he was constantly getting himself into trouble.
^ all pirates in PoC lived by the "pirate's code"
^ that as Jack Sparrow liked to remind everyone 
~~ They're really more like 'guidelines'~~

^ some of these are really just helpful tips for contextualizing android if you're coming from a different programming ecosystem

-- 

# Where did these come from?

^ so i started at Electric Objects in January of this year
^ and started where, i'm sure most of us end up every once in a while:
^ with this burning question

---

# HOW DO I START AN ANDROID APP?

^ um.  good question.
^ # awkward pause #

--- 

~~ image of Electric Objects Mock Ups ~~

^ if you're lucky, you probably start off with something that looks like this.
^ mock ups that your designer made.
^ if you're not lucky, you'll definitely start out with something that looks like this

---

~~ image of iOS app Mock Ups ~~

^ mock ups for an iOS app.




--- 
GUDING PRINCIPLES:
- Broken apps are bad (no one likes them)
- Use shit that works
- Maintenance costs aren't so much as painful as just fucking annoying. Minimize maintenance costs.
- Less Testing is More Testing (write less code*)
- Android Manufacturers break enough shit without my help  (Samsung Logo)
- App features are not set in stone : corollary, make it easy to make your designers happy.

* to make my point about how guideliney these are, I'm going to contradict this in a second

^ so here's my list of Android App Developer's Code
~~ A Developer's Code* ~~ (do the cross out the text thing here!)`
* More like Guidelines

-> Avoid Fragments (for view code)
 - Broken apps are bad
 - Minimize maintenance costs
 - Less Testing is More Testing
 - Use shit that works
-> There is a one to one correspondence between a wireframe and an Activity
 - Make it easier to change shit
-> Don't retain instance state *LET YOUR ACTIVITIES BE FREE*
 - Less Testing is More Testing
 - Broken apps are bad
 - i18n shit
 - Android OEMs are a bunch of well-intending goons
-> Every Activity is a State Machine  ( this is mostly a mentality shift, or a way at looking at Android code, if you're coming from a different context)
 - Broken apps are bad
 - Use shit that works
 - Less testing is more testing
 ^ maybe you're a pretty advanced Android dev and you use Icicle to avoid having to think about this.
 ^ shame on you.
 ^ there are some really cool ways to make views retain their state for you. i wish i knew how to do this. cyril mottier has a pretty good talk (one of the few) on how to write views that retain state for you, which in my opinion is money.

-> Retain nothing but state (and hardware adapters)
 - Broken apps are bad
 - Use shit that works
 - Less testing is more testing

-> One that I wish I had added: Favor Composition over Inheritance (for Activities)

~~ show off all of our activities. we have so many activities. but they're tiny ~~

--





** VACATION THAT SETS YOU FREE **
** that's a lot of responsability for a vacation, dont you think? **
** YES BUT FREEDOMMMMM **
** reality lisa, be realistic.  what's going to make this vacation different than the other ones youve taken? **
** how is spending time trotting around Europe really any different than trotting around NYC? **
** WEIRD LANGUAGES, DIFFERENT SYSTEMS, BEAUTIFUL BEACHES **

if you are a prolific person, then be prolifical

awkwardness is no excuse for lack of manners.
no manners is a fear of interacting.  COWARDICE!?

govind is a very well mannered person.  is it a matter of remembering to interact?
of interacting at the right level? of saying hello, how are you doing?

today, i learned that six thirty is family dinnertime at the neighborhood diner.

i dont want to couch my view as the 'feminist' perspective, but thats what it is.
at the root core of it. a feminine perspective.
rock it girl

if my legacy had to be anything, i think i would want it to be coinage of the term 'dainty hiking'
attributes of a 'dainty hiker':
really into ultralight packs (this is a practical concern, a dainty hiker is weak as shit)
never seems winded
bounds down hill
shoes that look more like slippers than hiking boots (see above about being weak as shit. those hiking boots are heavy as fuck)
always seems to have all the things that you could need.
crosses streams via the rocks whenever possible (dislikes wet feet)
takes a lot of pride in finding convenient spots to pee
first person unpacked at camp, first person packed in the morning
