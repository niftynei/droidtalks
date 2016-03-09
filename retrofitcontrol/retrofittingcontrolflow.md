
#Retrofitting Control **Flow**

^ This talk today is about a lot of things.

---

## Retrofit

^ on the face of things, it's about retrofit, a popular annotations-based Java library for Android that manages the composition of API calls and marshaling of those responses into POJOs for you. as a library, Retrofit's quite elegant.

---

##Control Flow

^ if you dig down a bit deeper (and by that, i mean read the second set of words on the talk title), you'll see that this talk is also about control flow.  specifically how do decisions about error handling impact control flow.

---

##Library Maintainer Decisions' Trickle-Down Impact

^ and last, and least obviously, it's about how decisions that library maintainers impact the architectural decisions of consumers of those libraries, downstream devs like you. and like me.

---

# __Background__

^ Let's start with a little background.  How many of you here today have ever used Retrofit?  And how many of you are using the latest version, 2.0+?  Still on 1.x?  Uncertain?

---

# __Background__

^ For those of you who aren't in the know, Retrofit v2 was released sometime last year, around the time of DroidCon NYC (August).  It's been through a few beta releases, is currently in beta4, pending an update to its multi-part APIs.  

^ The update to v2 was a breaking change, the project maintainers (ahem) were very forthcoming with this fact.  I'd like to focus on the change that had the biggest impact for my application: the way error handling is handled.

---

# __Handling Errors v1__

^ Prior to v1, when an error ocurred, Retrofit would throw an error, called a RetrofitError

---

```java
try {
  request.makeRequest();
  // if you reach this, request succeeded!
  return request;
} catch (RetrofitError error) {
  Response errorResponse = error.getResponse();
  switch (error.getKind()) {
    case NETWORK: 
    case HTTP: 
    case CONVERSION: 
    case UNEXPECTED:
     // deal with error
}
```

^ an error could be thrown for a bunch of different reasons.

---


- NETWORK
- HTTP 
- CONVERSION
- UNEXPECTED

^ a network error meant something went wrong w/ SSL or the device is offline
^ HTTP was a problem with the server, ie you successfully reached a server and received a response but it was something outside of a 200 or a 300 range

^ a conversion error meant that the JSON you were expecting and the JSON you received were structured differently (ie your conversion library choked)
^ and unexpected is for anything that didn't fall in any of the other categories. whoops.

^ There were some downsides to this. 

^ For successful calls, it made it near impossible to get any meta-data about the call (timing, headers, actual response code).

---

^ the nice side though, it let you write code like this

```java
void Boolean onMakeRequest() {
  List<Shoes> shoes = // call to get shoes
  return shoes.isEmpty();
}
```

^ Notice the amazing lack of error checking.  If something went wrong, I don't have to worry about it.  Retrofit will escape from this block for me, and my error handling code will appropriately do whatever it needs to do to signal this back to the original caller.

---

^ This is particularly handy if you want to make more than one call.

^ Note that you should only do this if you're *NOT* doing immutable operations with the server.  Like, nothing that you can't roll back from, since it will be unclear where in the chain things went wrong.

```java
void List<Shoe> onMakeRequest() {
  List<Shoe> myFavoriteShoes = // call to get my shoes
  List<Shoe> friendsFavoriteShoes = // call to get friend's shoes
  return myFavoriteShoes.addAll(friendsFavoriteShoes);
}
```

---

^ or at least.  that's how my code used to work.  retrofit 2 came out and the whole world changed.

---

# __Handling Errors v2__

---

```java
try {
  response = request.makeRequest();
  if (response.isSuccess()) {
    return request;
   } else {
     // server error?
  }
} catch (IOException e) { // network error
} catch (Exception ex) { // unexpected/parse error
  throw ex;
}
```

^ this is what my error handling block looks like now with v2 of retrofit.

^ you'll notice the case statement is gone, and instead there are a few different types of error blocks, one that catches the network errors (IO) and unexpected errors.

---

V1 Success != V2 Success

^ this is because Retrofit 2 redefined what constituted a "successful" call. 

---

## v1 Success:
- connected to the server
- received a successful response
- parsed the response successfully
- here's your object.

---

## v2 Success: 
- connected to the server
- received *a* response.

---

^ and my fancy multi-call blocks had to do some more checking. :/

```java
void List<Shoe> onMakeRequest() {
  Response shoesResponse = // call to get my shoes
  if (!shoesResponse.isSuccess()) return error!!;

  Response friendsResponse = // call to get friend's shoes
  if (!friendResponse.isSuccess()) return error!!;

  // otherwise proceed.
}
```

---

# __Try/Catch Blocks__

^ So this change brings up a *really* interesting point about programming in java.

^ Java is a strongly typed language with some generical capabilities.  Typically this means that a function can only return one type of object, and you have to know that object's type at compile time.

---

1 method call -> 1 object returned

^ put another way 1 method call -> 1 object returned.

^ but try catch blocks allow you to sort of get around this restriction.
^ or at least, if you use them the way that Retrofit v1 did, they certainly do.

^ in V1 of retrofit, i could assume complete success, and accurately ask for an expected type in my network call libraries, because the try/catch block assured that execution would stop before the invalid code was reached.

^ which is kind of like letting code return more than one object type. 

---

1 method call -> 1 object type || ∞ errors

^ so a better way to say this is that 1 method can return 1 good object type, and an infinite (almost) number of errors.

---

# __Retrofit v2: but why?__

^ given my presentation up to this point you might be wondering why did the library maintainers move away from this format of error catching. 

^ a good question i asked myself was: what were the project maintainers optimizing for? i strongly believe it was not the use case that I was employing Retrofit in.

^ There was one other change that was made as part of v2 that I think answers this question, and offers a little explanation for the "why" this change was made in the first place.

---

```java
\\ v1
@GET("/my/shoes")
List<Shoe> getShoes();

\\ v2 
@GET("/my/shoes")
Call<List<Shoe>> getShoes();
```

^ in v2 of retrofit, they condensed the API for a service down to a single object type: a `call` object. this change simplified the code need to write asynchronous callbacks, instead of needing a different method entry for every type of object caller (RxJava, etc), now there's only 1 method you need in your Service spec.

^ as a biased observer, this change greatly benefitted projects that use asynchronous callback frameworks for their network requests.  projects that don't ended up with a bit more of a wash.

---

# __Retrofit!!__
## The Best API Glue On The Market!

^ as it stands, Retrofit remains the best API to POJO glue available for Android. these changes just made a great case study about the relative merits of error handling.

---

## Thanks!

#### @niftynei

^ I'd like to thank Square Eng for inviting me here today. It's rare that you get the opportunity to openly critique a library in a forum hosted by the organization that sponsors its development.  So, yes thank you very much for inviting me to come speak today.
^ And thanks to you all for being a great audience. If you've got questions about anything I talked about today, hit me up on Twitter at @niftynei
