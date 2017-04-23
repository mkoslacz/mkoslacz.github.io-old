You're probably wondering why I did such stuff:

```Java

createSomeStream()
    ?.doOnNext { doStuffIWantOnTheEndOfStream() }
    ?.doOnError { handleError(it) }
    ?.retry()
    ?.subscribe()
```

instead of:

```Java

createSomeStream()
    ?.subscribe(
        { doStuffIWantOnTheEndOfStream() },
        { handleError(it) })
```

Well, after the error event Observable finishes its lifecycle - it unsubscribes all its observers. We don't want this behavior. If user wants to log in and he's out of the Internet, he will get some error about it. Then, if he manages to get some network and retries this action, nothing will happen, as our Presenter unsubscribed from login button events. That's nasty. We don't want that, so that's why we use side-effect operators and retry. Actually, we could do it like this:

```Java

createSomeStream()
    ?.doOnError { handleError(it) }
    ?.retry()
    ?.subscribe { doStuffIWantOnTheEndOfStream() }
```

This will cause an error in `doStuffIWantOnTheEndOfStream()` not to be passed to our doOnError and retry, what will lead to crash in RxJava 1.x and swallowing (delegating to the default error handler more precisely) an error and unsubscribing in RxJava 2.x. Well, yeah, it looks like a long RxJava story is starting right now, so I'm going to stop and give you [this official link](https://github.com/ReactiveX/RxJava/wiki/What's-different-in-2.0#error-handling) ;). We could add some error handler to subscribe method to notify user about the error in the view (in our case) and ie close the screen or an app, but our production experience in Movibe / Wirtualna Polska shows that handling unexpected view errors in such way allows us to get rid of many annoying, exotic-device-specific crashes in ie. TextViews. I know that you got used to hearing that your app shall crash in unexpected scenarios but actually our user will get an "whoops, something went wrong" toast in such scenario and he can just tap the button once again, and in all of the cases of random errors we have seen it worked at the second time, what's much faster and more friendly to user than crashing, launching app again, navigating to given screen and putting all of the data there again. We report all of doOnError caught errors to Fabric anyway, so if something really doesn't work we will realize that quickly, and user can go on with using other app functionalities without being kicked out of the app. That's why we have introduced the `retrySubscribe(...)` method that allows you do it using one call:

```Java
fun <T> Observable<T>.retrySubscribe(onNext: (T) -> Unit,
                                     onError: (Throwable) -> Unit) =
        this.doOnNext(onNext)
                .doOnError(onError)
                .retry()
                .subscribe()!!
```

So let's put it to the separate `ObservableExtensions.kt` file in the `util` package and replace the mentioned calls using this metod:
