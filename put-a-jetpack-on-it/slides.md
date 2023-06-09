# Putting a Jetpack on your legacy codebase

## Kurt Nelson (he/him)
### Android Platform at Pinterest

Hey everyone, Kurt from the Pinterest Android Platform (or Android Experience or AndroidX) team. We actually call ourselves the AndroidX team internally and we claimed it first!

---
What do I work on?

# Platform Teams
	- Mobile-wide issues
	- Build, framework, testing
	- Android, iOS, Web

At many of the bigger tech companies with longstanding big apps, we spin up platform and infrastructure teams within engineering to concentrate on mobile-wide issues. Sister teams of ours for example are the mobile builds team and the test tooling team. There are also equivalents for iOS, Web, etc.

---
# Forward-looking big picture

### Ensuring the stability, security and performance going forward

### Library choices and upgrades

One of our roles is to look at both the big picture and the future at the same time, hopefully heading external changes in best practices off enough that feature teams can write features and not have to worry about things like long term stability, performance, and library upgrades.

---
# We experiment early with new technologies

This means we are also in charge of trying out new libraries, best practices, and patterns to see if they belong in our app.

---
# We pay attention to what is going on in the Android world
	hi room.

Platform teams are often in the room early, even possibly getting first access to libraries to play with.

---
# Handling Android Changes

Of course, when Google makes large changes, we don’t as much have a choice of if we will use it but of when we will pick up the changes. 

---
### Handling Android Changes
# Kotlin

Kotlin was a great example of this- At Pinterest, we were pretty early to stamp it as generally available in our codebase. However over at Uber, it was causing build performance regressions annoying enough that it was just marked as GA recently in their code base.

---
### Handling Android Changes
# Jetpack Compose

On the flip side, we still have not given Jetpack Compose the platform team blessing as we see meaningful regressions in the performance of our home feed when we run A/B tests. 

---
### Handling Android Changes
# Jetpack Compose
	See the panel from yesterday with my teammate Christina Lee!
We want to allow developers to use Compose soon, but are going to have to wait for Google to finish some performance improvements first before we can have open season for converting features.

---
# Knowledge transfer
### Getting 100+ engineers to hold it right.


---
### 	At a large organization, we need to teach about it, roll out access, review new usage, and handle deprecation.

Beyond just the code itself, at a large organization we have to figure out how to roll out the knowledge on using the technology to 100+ engineers, how to make sure we don’t back slide, and when we finally say, “no, you can’t do that anymore”

---
# Migration
## We need to make sure it is safe to swap code out.
A migration plan needs creating: do teams have to migrate their call sites, do we just deprecate it in place, or can we centrally hot swap it out, preferably under experiment.

---
# Secret history lesson time

I happened to work at Google around the time Jetpack came to be.
---
# While you were using ALL the Square libraries...

We basically had our own internal versions of common things such as OkHttp and a fork of Glide inside Google. A good chunk of this was because Google’s stack is heavy on protocol buffers instead of JSON.

---
# Google has Product Areas
### Android and the Google suite of apps historically only shared the CEO.

Inside Google, the company is divided up to Product Areas, or PAs. Android is its own PA while the main suite of Android apps from Google are sprinkled about.

---
# Google Plus was chunked up and turned into the internal framework.

Around the time that Google Plus was being built, the company realized that maybe it was time to at least have a standard way of doing the Google apps. The team that worked on Plus turned into the Frameworks: Android team and became in charge of architecture for apps within the main monorepo at Google, google3.

---
# Hilt and Jetpack Datastore came from G+

I joined shortly after and started in on things such as the predecessor to Jetpack datastore, proto datastore.

---
# Frameworks even at Google were a fragmented mess.

The “legacy” internal apps each did their own thing. G+ plus and newer finally started to share code.

---
# Along Comes Jetpack…

While we were trying to fix our mess just across the Google suite of apps, the Android Product Area folks also decide the external world is messy.

---
# Original goal was to decouple app behavior from the OS
### Hence Jetifier.

SharedPreferences vs DataStore is a prime example: SharedPrefs had implementation detail differences depending on the OS version, but wasn’t doing anything the app couldn’t do on its own.

---
# It was a piecemeal migration for the Google suite of apps.

For Googlers, there was way less of a specific “we are doing jetpack!” moment, it was adopted app by app if they weren’t already sitting on top of the framework teams’ tools.

---
## Spoiler:
# We took care of users of the G+ framework

---
# Back at Pinterest
	- The app is from around 2012
	- Over 700 unique committers
	- Most code is O(years) old

The Pinterest app dates from around 2012, having had over 703 committers in the repo and the average age of checked in code of old.

---
# One app, one repo
	100 monthly committers

Luckily we only have one major app in a not-crazy-big repo. (Uber has had over 10 active android apps in one monorepo) Still, we have about 100 developers committing to the repo every month.


---
# The Birth of ScreenManager
### Our attempt to fix fragments

Before Jetpack, engineers at Pinterest had to do something. Fragments were still often extremely painful and error prone, with commitAllowingStateLoss starting to be way too common. Enter sometime in 2016: ScreenManager.

---
# Screen API
	Our innocent looking Fragment replacement.
```kotlin
    fun bind(screenDescription: ScreenDescription)
    val isScreenBound: Boolean
    val wasScreenEverBound: Boolean

    fun activate()
    val isScreenActivated: Boolean

    fun deactivate()
    fun unbind()
    fun destroy()

    fun getView(): View?
```
Look at this API, so innocent. It has no constructor parameters, has a method to setup the content, then a method to say “hey, you’re on screen!”. Teardown is symmetric.
---
# Appropriate at the time
	- Our app is full pages
	- MVPish
	- Solved fragment transaction crashes
At the time, it was the right thing to do! Fragments were overkill especially considering the app is primarily full pages, no need for multiple screens at once. This API lined up with MVP well, which at the time was probably the most common way to architect an app.

---
# The Gotcha
Our problem was in the way we bound our Screens. You don’t see immediately from the API the definition for `ScreenDescription`:
```kotlin
interface ScreenDescription : Parcelable {
    val screenClass: Class<out Screen>
    val screenTransitionId: Int
    val arguments: Bundle
    var instanceState: Bundle?
    var showBottomNav: Boolean
    var uniqueId: String
    val results: MutableMap<String, Bundle>
    val internalName: String
}
```

Look at that code for a second and think about how `bind` might work and see how we got ourselves in a pickle: We’re using a `Class` to identify the type this thing is, and you know what that means the implementation is!

---
# Reflection!
#### And basically our own version of a FragmentManager
#### And our own version of Navigation

On its own, this really wasn’t bad. FragmentManager itself does similar things with Class-lookup map itself to handle transactions and such. The argument bundle is also in this parcelable description and not part of the construction of a Screen, so it is not available as early as in a proper fragment.

---
# How do we back out?
## And what do we back out first?
Now that Fragments are happier and work well with Jetpack Nav, plus eventually we want to use Compose, we’ve got to figure out how to back ourselves out of this design decision.
---
# Prioritization
---

### Prioritization
# Which First?
	- Jetpack Navigation
	- Jetpack DataStore
	- Real Fragments & Lifecycle
	- Hilt
	- Compose
	- ViewPager2
We know we want to use Jetpack Navigation, DataStore, normal fragments, Hilt and friends, plus set ourselves up for Compose when it is ready for our use case. There’s a ton of other little knock-on effects, our ViewPagers are slightly off since they need to host Screens for example. 

---
### Prioritization
# Tradeoffs
	- Reliability
	- Rollout speed
	- Unknowns in our legacy code
	- Learning
	- Normal day to day work
And of course we need to do this all safely- hopefully without really having to do anything other than “hey, you can use the new shiny now!” to feature engineers. Any library that uses the AndroidX Lifecycle class is likely partially and mysteriously broken when running inside of our hacked fragments.

---
# Legacy Async Code
### We *just* made coroutines GA
We’re also just now introducing coroutines, so there is yet another factor to consider, when do we leave RxJava2 that makes you cry a little inside alone when migrating around it?

---
All of these discussed items are true for any library upgrade or addition but
# Jetpack Libraries are scarier than others as they are crucial to showing anything on screen.

Certain library changes and mutations over the years have been pretty drop in place, think image loaders: they basically all take in one line a context, what to load, and where to put it. Even Dagger to Hilt isn’t bad as Hilt is set on top of Dagger, you don’t have to do the entire app in one pull request.

---
# Outsourcing from your team
## What can others do?
Either to feature teams or contractors if you have them, what turn-key migrations can someone else do without needing the full context? 

---
#### Outsourcing from your team
# Java to Kotlin
For our lingering Java to Kotlin for example, we turned on NullAway and then had an external resource perform the very rote task of removing suppressions and fixing the nullability there. Now when someone wants to convert from Java to Kotlin, it is a much safer process. This is still an ongoing project, we are not at 100% Kotlin especially in our “dangerous” framework code.

---
# Migration Options
From minimal to maximum disruption
---
#### Migration Options
# Only for new code
No expectation to adjust any legacy code. For most people, Jetpack Nav would be a good example here, it is easy to onboard to, you just inform everyone “hey, add new features to the nav tree!” and run with it.
This of course is foiled if you already have your own similar navigation framework that...runs on reflection. We have to migrate whole sections of features together that have argument dependencies.

---
#### Migration Options
# Under the hood
Great for when the feature-exposed behavior does not change. As long as you can maintain a consistent interface, you can then A/B test implementations without any disruption to users or developers. A prime example of this is us comparing image loading libraries. We will be doing this to go from SharedPreferences to DataStore while allowing an invisible data migration phase.

---
#### Migration Options
# Shim layer
This is how we are handling coroutines, exposing a new coroutine native API and then putting the bridge code underneath. As features decide to use coroutines, they will be able to query all of our repositories and models only thinking in coroutines while our complicated legacy Rx code underneath still does the heavy lifting. Eventually we will do an invisible under the hood swap to pure coroutines at our own pace.

---
#### Migration Options
# Central migration
We have fully executed this approach for Dagger -> Hilt. A few infra devs were able to put out pull requests over a period of months moving Gradle modules over with minimal disruption to feature developers. Likely this will be the method we take for getting on Jetpack Navigation.

---
#### Migration Options
# Distributed self-serve
This is where teams are encouraged to at their own pace to manipulate their own code, but with an expectation to do so in a reasonable time. Likely once the list is burned down, a platform team member will take over migrating any abandoned code to allow a full retirement of the old method. A project manager is likely involved at this point for tracking.

---
#### Migration Options
# Distributed mandate
This is the worst one. “Migrate all your code by date X or executives will chase you down.” Mandates having someone project managing, and likely requires approval from far above the infra team. The main example here is a security related issue or a business threat. This is exactly what we hope to avoid by getting off of legacy systems in the first place.

---
# Where are we now?
---

# Full Hilt
## Early 2023
---
# Jetpack DataStore
## In progress
---
# Compose
## Blocked on performance improvements
---
# Lifecycle
## Using normal androidx.Lifecycle patterns
But shimming it underneath
---
# The rest?
## Maybe I’ll tell you next DroidCon

---
# Putting a Jetpack on your legacy codebase
## Kurt Nelson (he/him)
### Android Platform at Pinterest
	Mastodon: kurt@nelson.fun, kurtisnelson on the socials
	 [github.com/kurtisnelson/presentations](https://github.com/kurtisnelson/presentations/)