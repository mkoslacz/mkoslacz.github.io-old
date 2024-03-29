---
layout: post
title: "Moviper - the Android VIPER Library"
date: "2017-02-08 08:38:09 +0100"
---

# Introduction

On the previous post I have covered pros and cons and a general idea of the Viper architecture. For now let's focus on [Moviper - the Android Viper library](https://github.com/mkoslacz/Moviper). Moviper comes in many different flavors, there are more and less advanced ones - you can just pick one that fits you most. In this post I'll show you the flavor I use in the daily basis on the example of a simple Github login screen. I use Rx submodules Passive Autoinject Views and I wrap it up using Kotlin language with Android Kotlin Extensions. Sounds scary? Actually it's pretty straightforward. Fear not and read on!

We are going to create very simple app, that will allow a user to log in to Github and go to the profile page, and provide info about the app.

It will consist of
- login screen:
  - allows user to go to the help screen,
  - allows user to login and then leads to profile page.
- fake profile page:
  - displays logged in user username.
- fake help page:
  - displays "help" text.

  [//]: # (3 images side-by-side table)

  |Login screen|Fake profile screen|Fake help screen|
  |:----:|:----:|:----:|
  |![Login screen.]({{ site.url }}/assets/LoginActivityLayout.png){:.center-image }|![Fake profile screen.]({{ site.url }}/assets/ProfileActivityLayout.png){:.center-image}|![Fake help screen.]({{ site.url }}/assets/HelpActivityLayout.png){:.center-image }|

  *This is our sample app mockup. As you can see it's not a UI tutorial ;).*{:.image-caption}

Ok, enough talking, let's get our feet wet!

# Setup

First of all, install the [Moviper Templates Generator](https://github.com/mkoslacz/MoviperTemplateGenerator) using instructions provided in the repository readme. Just click the link. Remember to restart the Android Studio after the installation!

After that, start up with a brand new Android project with no Activity. If you're new to Android you can follow a official Google tutorial [here](https://developer.android.com/studio/projects/create-project.html) to learn more about stuff we will discuss here.

The next step is pretty obvious, go to [Moviper Github page](https://github.com/mkoslacz/Moviper) and find the latest Moviper version and import its Rx-dependency adding following line to app module gradle dependencies and sync the project.

```(groovy)
dependencies {
  compile 'com.mateuszkoslacz.moviper:moviper-rx:2.0.3'
}
```

Now things get interesting. Let's create our first Viper Activity! Right-click the app package and select `New -> Moviper -> RxActivity`.

![This is where lies RxActivity generator.]({{ site.url }}/assets/chosingViperRxActivity.png){:.center-image}
*This is where lies RxActivity generator.*{:.image-caption}

*Pro tip: you can even assign a shortcut to creating any of the Moviper modules using `Manage shortcuts` menu (easy to find using our standard `cmd/ctrl + shift + A` magic) and searching for "moviper" actions and right clicking them to add a shortcut. That will be even more smooth using the great [Key Promoter](https://plugins.jetbrains.com/plugin/4455-key-promoter) plugin, it will suggest you assigning a shortcut to the most used actions, you should definitely check it out!*{: style="color: gray"}

![Assigning shortcuts for Moviper actions will definitely come in handy.]({{ site.url }}/assets/assignMoviperShortcuts.png){:.center-image}
*Assigning shortcuts for Moviper actions will definitely come in handy.*{:.image-caption}

Fill up the form like I did on the screenshot below: name it `LoginActivity`, mark it as a `Launcher Activity` and select a `Passive Autoinject` type. There are some more options there and I hope that they're pretty self explaining. If I'm wrong - just trust me for now - I'll cover them later anyway.

![That's how I have created the LoginActivity.]({{ site.url }}/assets/ViperActivityCreation.png){:.center-image}
*That's how I have created the LoginActivity.*{:.image-caption}

After confirming our choice we land in the `Contract` interface. At your left hand side, on the project overview, you can see the rest of the classes generated by plugin in `*.viper.login` package. Viper is a general package for viper modules, and login was inferred from the name we gave the generated Activity.

But hold on - I told you that we will make a Kotlin app, and these files feel somehow suspicious. Yep, MoviperTemplateGenerator creates only Java file sets and for now there is no plan to migrate it to Kotlin. This way we can use it in for generating Java code, and for Kotlin apps we can convert it to Kotlin very easy, so let's do it right now using magic `alt + cmd/ctrl + shift + K` shortcut at whole `login` package.

For now we have a set of 5 Kotlin files:

- `LoginActivity` class (the view)
- `LoginContract` interface
- `LoginInteractor` class
- `LoginPresenter` class
- `LoginRouting` class

It's a good time to stop now and discuss the responsibility distribution over these components. In a classical objc-viper we have a following division:

>The main parts of VIPER are:
>
>View: displays what it is told to by the Presenter and relays user input back to the Presenter.
>Interactor: contains the business logic as specified by a use case.
>Presenter: contains view logic for preparing content for display (as received from the Interactor) and for reacting to user inputs (by requesting new data from the Interactor).
>Entity: contains basic model objects used by the Interactor.
>Routing: contains navigation logic for describing which screens are shown in which order.

*Source: https://www.objc.io/issues/13-architecture/viper/*{: style="text-align: center; display: block; font-size: 80%;"}

For Moviper I have worked out the following, slightly shifted responsibility fragmentation:

- View: displays what the Presenter wants and relays user input back to the Presenter. (exactly the same as above)
- Interactor: contains the data read/write and preprocessing logic - all of the api and db calls go here.
- Presenter: contains the buisness logic: reacts to user input, ie translating it to the interactor and routing calls, decides what to display on the view, validates the data in the business usecases sense (ie username can't be empty).
- Entity: data objects that are used in the app business logic, independent from a data handling and view implementation.
- Routing: contains all system-related logic. You can alternatively call it "System". It manages navigation, screen transitions, scheduling notifications, etc.

Mentioned division allows us to create modular, testable, clean and neat code. Now we know where to put our stuff, so let's begin to code!

*Important! Moviper allows you to handle orientation changes in the very easy way and publish results from background thread after recreating view but now, for simplicity, please mark our activity to be in a blocked, portrait orientation by adding the `android:screenOrientation="portrait"` line to its entry in the `AndroidManifest.xml` file as follows:*

```xml
<activity android:name=".viper.login.LoginActivity"
        android:screenOrientation="portrait">
    <intent-filter>
        <action android:name="android.intent.action.MAIN"/>

        <category android:name="android.intent.category.LAUNCHER"/>
    </intent-filter>
</activity>
```

*Handling orientation changes in Moviper will be covered in the later posts*

# Contract

The starting point of every screen is a contract. It generally has the one purpose - it defines whole control and data flow in the given screen. Let's define methods needed to implement the whole login screen:

```Java
interface LoginContract {

    interface View : MvpView {
        val loginClicks: Observable<LoginBundle>
        val helpClicks: Observable<Any>
        fun showLoading()
        fun showError(error: Throwable)
    }

    interface Interactor : ViperRxInteractor {
        fun performLogin(loginBundle: LoginBundle): Single<UserModel>
    }

    interface Routing : ViperRxRouting<Activity> {
        fun goToHelpScreen()
        fun goToProfileScreen(user: UserModel)
        fun finish()
    }
}
```

I defined here some data classes to describe content we will use in the created methods. `LoginBundle` data class is used as a Observable event type that will, well, bundle the user login data from view:

```Java
data class LoginBundle(val login: String,
                       val password: String)
```

And there is also a `UserModel` class that will represent our user retrieved from the remote. It is an equivalent of a Github api json returned of an endpoint we will use:

```Java
data class UserModel(val login: String,
                     val id: Int)
```
Let's put them to the `data` package:

![Let's put our nice data classes to the separate package.]({{ site.url }}/assets/DataClassesLocation.png){:.center-image}
*Let's put our nice data classes to the separate package.*{:.image-caption}

Going back to our contract - as you can see, presenter has no interface here. Welcome to the passive word from the chosen Moviper flavor. Passive Viper means that View has no idea about Presenter is being attached to it. Actually you can access presenter from view using `presenter` property / `getPresenter()` method, but in this flavor it will be just plain ViperPresenter, so you won't have an access to your actual presenter methods. View communicates with presenter through event streams exposed through view interface to which presenter subscribes when attaching to the view.

Interactor and Routing are, let's say, Presenters "tools", presenter delegates work to them and receives results using Observables. That said, there is no component that calls Presenters methods, so there is no need to make it implement any interface. This allows us to create multiple presenters (where each has it's own routing and interactor) for one view and switch them seamlessly! We'll go back to this feature later.

# Presenter

Ok, so let's begin implementing the Presenter. Let's check out what we have there already.

```Java
class LoginPresenter :
        BaseRxPresenter<LoginContract.View,
                LoginContract.Interactor,
                LoginContract.Routing>(),
        ViperPresenter<LoginContract.View> {

    override fun createRouting() = LoginRouting()

    override fun createInteractor() = LoginInteractor()
}
```

As you can see, there are declarations of routing and interactor generated by generator already. You probably noticed that class declaration is pretty complicated because of some crazy generic stuff. Don't worry, after implementing some Viper modules you will get what happens here, but for now just trust the force (the generator, to be more specific).

Let's begin with defining what shall happen on the very beginning of the presenter lifecycle in Viper module - on a presenter to a view attach. To achieve that override the `attachView(attachingView: LoginContract.View?)` method. Don't forget to call super to allow Moviper do its setup work and to use `view` property, not the `attachingView` argument as a rule of the thumb! (the latter is necessary for correct orientation changes handling, we will cover it later)

```Java
class LoginPresenter :
        BaseRxPresenter<LoginContract.View,
                LoginContract.Interactor,
                LoginContract.Routing>(),
        ViperPresenter<LoginContract.View> {

    override fun attachView(attachingView: LoginContract.View?) {
        super.attachView(attachingView)

        addSubscription(
                view
                        ?.loginClicks
                        ?.doOnNext { view?.showLoading() }
                        ?.observeOn(Schedulers.io())
                        ?.flatMapSingle { interactor.performLogin(it) }
                        ?.observeOn(AndroidSchedulers.mainThread())
                        ?.retrySubscribe(
                                {
                                    routing.goToProfileScreen(it)
                                    routing.finish()
                                },
                                { view?.showError(it) }))


        addSubscription(
                view
                        ?.helpClicks
                        ?.retrySubscribe(
                                { routing.goToHelpScreen() },
                                { view?.showError(it) }))
    }

    override fun createRouting() = LoginRouting()

    override fun createInteractor() = LoginInteractor()
}
```

And define our special `retrySubscribe` operator in `ObservableExtensions.kt` file in `*.util` package:

```Java
fun <T> Observable<T>.retrySubscribe(onNext: (T) -> Unit,
                                     onError: (Throwable) -> Unit) =
        this.doOnNext(onNext)
                .doOnError(onError)
                .retry()
                .subscribe()!!
```

This operator allows our streams to signal an error and do not unsubscribe after that - if user wants to log in and he's out of the Internet, he will get some error about it, and we want to allow him to retry this action. I'll describe implementation details of it in the another post.

Now let's go back to our presenter. As you probably noticed, presenter has an access to view, routing and interactor in a whole class scope. Moreover, it automagically cast it to the appropriate interface defined in a contract. Note that I always use the optional view call - it's just easier to do it in a no-brainer way. As we often use thread switching in our streams, view could get detached from the presenter in the meanwhile. Don't let Android Studio fool you using "unnecessary safe call" message! On the other hand Interactor and Routing are tightly coupled with presenter so there is no need to use safe calls on them in any situation.

The other fancy thing there is that we haven't even touched the another components but we can safely implement whole presenter! Using Moviper you can pararellize the work on the single module at whole team. Making work more focused on single screen makes you send new builds to QA faster, so there is no bottlenecking on the very end of the scrum sprint!

On the example you can see that each stream is wrapped in a `addSubscription` call. It's one of the built-in Moviper methods that allows you forget about the need of unsubscribing your streams to avoid memory leaks. Just wrap all of your never completing streams using `addSubscription` and that's all! Your streams will be unsubscribed at presenter detach from View. If you have some objects in your presenter that need some special finishing, do it overriding `onDetach(Boolean retainInstance)` method (and once again, don't forget to call super!).

But remember, not every call of it means that presenter will be destroyed - if a `retainInstance` argument is `true` it means that attached view is destroying because of the Android orientation change, and presenter will be retained and reattached to the new view, so you don't have to nuke your delegates (at least these which don't use the view reference).

The thing worth emphasizing here is that your presenter won't be parceled and recreated or something like that, it will be the same, exact presenter that have been attached to the view before orientation change. Moreover, all of the background work that the presenter performs (and all of its children, with Routing and Interactor as a most notable example of them) will be delivered to the eventual view if the orientation change happens in the meanwhile if you follow one rule of the thumb - *always communicate with the view using the main thread* (http://hannesdorfmann.com/mosby/summary/ - *Can the Presenter and its view be out of sync during a screen orientation change?* paragraph).

Even if view will need to perform some hard work on the background thread, using the diffUtil for example - always call the view on the main thread and delegate the work to another thread in the view itself. For more info about the presenter and view lifecycle and behavior I recommend you reading the Mosby  http://hannesdorfmann.com/mosby/ docs and blog as the Moviper is built on top of this library, so it inherits the behavior of view-presenter relations.

Ok, so we have discussed the Moviper behaviour in this context, so now let's focus on our sample app logic:

```Java
addSubscription(
        view
                ?.loginClicks
                ?.doOnNext { view?.showLoading() }
                ?.subscribeOn(Schedulers.io())
                ?.flatMapSingle { interactor.performLogin(it) }
                ?.observeOn(AndroidSchedulers.mainThread())
                ?.retrySubscribe(
                        {
                            routing.goToProfileScreen(it)
                            routing.finish()
                        },
                        { view?.showError(it) }))
```
So let's see what happens here:
1. `addSubscription` call that wraps our stream to make sure that there won't be any memory leak
2.
3. call on the loginClicks stream provided by our view that is defined in our contract
4. we show the loading on the view to notify the user about the processing of the request
5. delegation of the work to the background thread from io scheduler as it will be network based
6. delegation of the data work to the interactor using a method that is defined in our contract
7. we switch the thread back to the mainThread as in the methods below we will touch the UI
8. we call our special operator that will allow us to retry the calls in case of failure
9.
10. if the interactor performed our request succesfully we delegate the screen switch action to the routing using a method that is defined in our contract
11. same as above but we finish login activity after login
12.
13. if the request has failed we show the error to the user

The second stream based on `view?.helpClicks` has the similar philosophy, so I won't cover it step by step as it's pretty straightforward in the context of the previous stream.

# Routing

OK, so our presenter is ready, now let's implement the routing. Let's start with using Android Studio auto-fix to implement stubbed methods from the contract:

```Java
class LoginRouting : BaseRxRouting<Activity>(), LoginContract.Routing {

    override fun goToHelpScreen() {
        TODO("not implemented")
    }

    override fun goToProfileScreen() {
        TODO("not implemented")
    }
}
```

To start these screens I will use starters here. Starter is a simple class that is used to, well, start the screens. But why to use them instead of just starting the activity here? There are two main reasons to do it:

1. We need to allow an user to go to two screens from this module (a profile screen and a help screen). As we create Viper modules in our app we will need to add six files (contract, Presenter, Interactor, View, Routing, layout) for each of screens and insert appropriate Activity declaration lines to AndroidManifest. Adding 12 files not related with screen we work at won't be a really good practice.
2. If we want to TDD our screen (and we do, I will cover it on the next blog post) it's much easier to mock the context and check if appropriate starters were called than test if appropriate activities were started. Moreover, it's much quicker to do it using mocked context than to use Robolectric to test that activities were started.

So let's create starters for mentioned screens and place them in correct packages:

![I put the starters for the separate packages in the same way I did with a whole Login Viper module.]({{ site.url }}/assets/StartersLocation.png){:.center-image}
*I put the starters for the separate packages in the same way I did with a whole Login Viper module.*{: style="text-align: center; display: block;"}

Ok, so now let's use our starters in routing:

```Java
class LoginRouting : BaseRxRouting<Activity>(), LoginContract.Routing {

    private val helpStarter = HelpStarter()
    private val profileStarter = ProfileStarter()

    override fun goToHelpScreen() {
        relatedContext?.let { helpStarter.start(it) }
    }

    override fun goToProfileScreen() {
        relatedContext?.let { profileStarter.start(it) }
    }
}

```

And a quick look how the starters look like:

```Java
class HelpStarter {

    // it won't work for now as we don't have HelpActivity yet, so for now let's leave this method empty.
    // fun start(context: Context) = context.startActivity(Intent(context, HelpActivity::class.java))
    fun start(context: Context) {}
}
```

As you can see, I have made them fields of our Routing. You will get the great benefit of it in the next post, while we'll redo the whole process using TDD. For now just trust me please ;). Take note of a `relatedContext` nullcheck using cool Kotlin [safe call](https://kotlinlang.org/docs/reference/null-safety.html#safe-calls) and [let function](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/let.html). In Routing it's important to always check if the mentioned context is attached, because if Presenter calls a routing on the non-main thread, it's possible that related context provider, ie. Activity already got destroyed, so it could lead to crash. Ok, so now we have implemented the routing despite that we don't have any other Viper module implemented, nice!

# Interactor

Ok, so let's implement the Interactor now, I did it that way:

```Java
class LoginInteractor : BaseRxInteractor(), LoginContract.Interactor {

    private val loginRepository = LoginRepository()

    override fun performLogin(loginBundle: LoginBundle) = loginRepository.performLogin(loginBundle)
}
```
With repository implementation delegated as a quasi-rdp ;):

```
class LoginRepository {

    private val retrofit = Retrofit.Builder()
            .baseUrl("https://api.github.com/")
            .addConverterFactory(GsonConverterFactory.create())
            .addCallAdapterFactory(RxJava2CallAdapterFactory.create())
            .build()!! // inferred

    fun performLogin(loginBundle: LoginBundle): Single<UserModel> {
        return retrofit
                .create(LoginAndGetUser::class.java)
                .getUser(loginBundle.authorization)
    }

    private interface LoginAndGetUser {
        @GET("/user")
        fun getUser(@Header("Authorization") authorization: String): Single<UserModel>
    }
}
```

Where I have extended LoginBundleClass like this:

```Java
data class LoginBundle(val login: String,
                       val password: String){

    val authorization = "Basic " + Base64.encodeToString(("$login:$password").toByteArray(), Base64.NO_WRAP)
}
```

*Pro reader note: yep, I know how it looks like, but dude, it's for presentation purposes.*

Well, I don't want to focus on networking implementation here, it's over-simplified moreover. I put it to delegate to don't distract you. Just trust me, this code takes the login and password and translates it in the way readable for server authorization. In the regular case we would need to keep the header for whole session using Interceptors, and I personally would abstract-out the network layer implementation from the Interactor using [Repository Design Pattern](https://medium.com/@krzychukosobudzki/repository-design-pattern-bc490b256006), but it's the material for an another article anyway. The important thing here is that we have our data handling delegated to Interactor.

# View

Now let's get the jump start to the view implementation and dive strictly to my ready implementation:

```Java
class LoginActivity : ViperAiPassiveActivity<LoginContract.View>(), LoginContract.View {

    override val loginClicks by lazy {
        RxView.clicks(loginBtn)
                .map {
                    LoginBundle(login = loginField.text.toString(),
                            password = passwordField.text.toString())
                }!!
    }

    override val helpClicks by lazy { RxView.clicks(helpBtn) }

    override fun showLoading() {
        progressBar.visible()
        loginBtn.gone()
    }

    override fun showError(error: Throwable) {
        progressBar.gone()
        loginBtn.visible()
        Toast.makeText(this, error.localizedMessage, Toast.LENGTH_SHORT).show()
    }

    override fun createPresenter() = LoginPresenter()

    override fun getLayoutId() = R.layout.activity_login
}
```

In this file I have used some cool Kotlin features to reduce boilerplate and make the code more readable. Firstly, there are [Kotlin Android Extensions](https://kotlinlang.org/docs/tutorials/android-plugin.html) that allows us reefer views using their ids directly in code without declaring them using `findViewById()` method. You can see that I reference progressBar, loginBtn etc directly, without any lookup and it works. The second thing is the use of [Extension Functions](https://kotlinlang.org/docs/reference/extensions.html#extension-functions) that I use to change the visibility of the views neatly, ie `progresBar.visible()`. I put view extensions to the separate file in the `utils` package:

`com.mateuszkoslacz.movipershowcase.util.ViewExtensions.kt`
```Java
fun View.visible() {
    this.visibility = View.VISIBLE
}

fun View.gone() {
    this.visibility = View.GONE
}
```

The third thing is a usage of [Kotlin lazy function](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/lazy.html) that allows us to initialize values that uses Android Views inside the class body, before views initialization, instead of `onCreate` method. Kotlin is sweet!

I've also used [Jake Wharton's RxBinding](https://github.com/JakeWharton/RxBinding) to easily create Rx streams from buttons.

And don't forget about the corresponding layout file. I won't include it, but you can check it out [here](https://github.com/mkoslacz/MoviperShowcase/blob/master/app/src/main/res/layout/activity_login.xml).

![This is how our view looks like.]({{ site.url }}/assets/MainActivityLayout.png){:.center-image}
*This is how our view looks like.*{:.image-caption}

As you can see, our view implementation is pretty straightforward. We show and hide appropriate views, show an error in the toast, and provide click streams - in case of login clicks, we map it to the `LoginBundle` using texts from inputs. There is also a defined presenter and layout, and that was done for us by MoviperTemplatesGenerator. In the view you just define the behavior, layout, and a presenter, and you don't have to worry about any kind of binding to presenter, handling lifecycle etc. Moviper does it for you.

It's worth noting that view is 100% passive here - it means that view is not aware of kind of presenter attached to it nor methods it has. It doesn't call presenter in any way - it just provides the interface to which a presenter can attach itself to. It's the presenter that decides what to do, what to use and what not to use - view has no app logic inside.

The clean design that makes our apps maintainable, but it also comes in handy in our production environment where we attach multiple presenters to our views using Moviper [ViperPresentersList](https://github.com/mkoslacz/Moviper#attaching-multiple-presenters-to-the-view). In Movibe / Wirtualna Polska we have a separate app logic presenter, analytics presenter, advertising presenter, where every presenter has its own routing and interactor crafted appropriately to the role of the given presenter. The fact that there are many presenters is transparent for the view. It allows us to dynamically attach or detach presenters to ie turn off ads for premium users.

Moreover, if your presenter has grown too much and for some reason you can't split your view to smaller chunks that corresponds with separate use-cases you can create the presenter for each use-case and attach the whole bunch of them to the single view. It allows us to keep our classes small, and smaller (in most cases) means more readable, more maintainable, more testable and more awesome. And still - with no changes to View!

It also works for defining multiple behaviors for one view. We use it ie to create a one login screen and reuse it in the various login methods as we allow authorization using some external account credentials in our apps. Moviper provides a ready-to-use tool for it called [PresentersDispatcher](https://github.com/mkoslacz/Moviper#choosing-presenter-on-runtime).

The idea works in both ways - we also swap the views using the same presenter and whole app logic, ie when we implement the Android TV version of our apps using the goodies from Leanback Library.

# Starters & Following Views

Ok, now our login screen is fully functional. But before launching it let's pretend that our sprint have just finished and we have to implement the next screens. Let's check out how it will influence our previous module (hint: it won't).

Let's create our HelpActivity that we're prepared to launch as we have created a starter for it before. It's not created as a Moviper Activity for simplicity, but it should be ;):

```Java
class HelpActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_help)
    }
}
```

and now we can uncomment the code we wrote before in our starter:

```Java
class HelpStarter {

    fun start(context: Context) = context.startActivity(Intent(context, HelpActivity::class.java))
}
```

What are the changes in the Login Viper code? None. The only difference is that our help button started to work. In Login Viper, what will be a overhead of an existence of a help screen if we decide to disable the help button using some remote config? Almost none (a one-liner HelpStarter class object created in Routing).

Moreover, we could remotely swap the starter to another one existing in our code to remotely modify the behavior of the app. Viper allows a very high modularity of apps in the level that allows to reconfigure the app on the fly. It's very helpful in handling special events (ie. special Christmas modules that can be activated and deactivated without an update) or some fatal scenarios in which some module malfunctions we can always disable it remotely to avoid further crashes during the hotfixing. More detailed article about this feature coming soon.

Now let's focus on the ProfileActivity, and once again,  we're prepared to launch it as we have created a starter for it before. It's not created as a Moviper Activity for simplicity, but it should be ;):

```Java
const val EXTRA_USERNAME_STRING = "EXTRA_USERNAME_STRING"

class ProfileStarter {

    fun start(context: Context, user: UserModel) {
        val starter = Intent(context, ProfileActivity::class.java)
        starter.putExtra(EXTRA_USERNAME_STRING, user.login)
        context.startActivity(starter)
    }

}
```

```Java
class ProfileActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_profile)
        usernameText.text = intent.getStringExtra(EXTRA_USERNAME_STRING) // display the username
    }
}
```

And once again, no changes in Login Viper code, and the only difference is that we're processed to the ProfileScreen after the successfull login.

Now our app is complete. We can put our the wrong credentials to our inputs and see the error, and then retry with correct ones and succeed. We can try login with network disabled and then retry after enabling. We can see that our progressbar shows while loading and disappears after the error, and our LoginActivity finishes after successful login. The help view is also functional. It's alive!

And well, it looks like we're finished our base example of Moviper PassiveView flavor in Kotlin. Of course, if you prefer Java you can use it as well, even better using some Moviper flavor that will replace Kotlin Android Extensions for you, ie Butterknife or Databinding one.

Moreover, if you don't like the passive view concept you can choose the flavor in which view is not passive and it calls presenters methods, or you can even pick the non-rx Moviper version. Feel free to choose your favorite! (but take note that I consider the flavor described in this post as a most boilerplate-reducing, readable, scalable and featureful) Just check out the repo.

# Sum up

To sum up let's take a look at our code:
- It's highly modular in way that allows us remotely enable, disable and swap modules and develop multiple screens at once without conflicts.
- The contract allows a dev to take a quick overview how the whole module works while each submodule is so simple and beautiful that it's understandable at a glance.
- The view is passive, so we can swap the presenters, split the presenter to multiple ones if it grows too much, or attach additional presenters for optional features.
- We wrapped all of our logic in Rx streams that allows us to track the app execution path easily and thanks to it it's almost completely crash-safe (not to be confused with being error-safe).

And there is a MoviperTemplatesGenerator that did all of the relations binding, class creating and inheritance defining for us! That's pretty nice bunch of features!

In this post series I have skipped some less important code, but you can check out and run [the whole sample here](https://github.com/mkoslacz/MoviperShowcase).

I encourage you to let me what you think about the architecture and the library itself. Feel free to contribute and report the issues.

And last but not least, stay tuned as I'll be posting more about the Moviper goodies:
- handling orientation changes with retaining background jobs state,
- Inter-Presenter-Communication,
- dispatching presenters on runtime (using the same view with different presetenters),
- using multiple presenters with one view,
- creating the viper modules for RecyclerView cells,
- reating Android Service based viper modules and standalone viper modules,  
- passing Intent extras to presenters,
- TDDing Viper modules,
- using Moviper Test utils,
- using iOS Moviper Version,
- sharing code between Android and iOS thanks to Moviper

And much more! Stay tuned!
