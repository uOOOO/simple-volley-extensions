# Simple Volley Extensions

[![Push Build](https://github.com/uOOOO/simple-volley-extensions/actions/workflows/push-build.yml/badge.svg?event=push)](https://github.com/uOOOO/simple-volley-extensions/actions/workflows/push-build.yml) [![Maven Central](https://maven-badges.herokuapp.com/maven-central/io.github.uoooo/simple-volley-extensions/badge.svg)](https://maven-badges.herokuapp.com/maven-central/io.github.uoooo/simple-volley-extensions)

## Download
```groovy
dependencies {
    implementation 'io.github.uoooo:simple-volley-extensions:0.1.4'
}
```

## Usage

### - Create GET request
```kotlin
RequestBuilder<String>()
            .setMethod(Request.Method.GET)
            .setBaseUrl("https://httpbin.org")
            .setRelativeUrl("/get")
            .setResponseParser(StringResponseParser())
            .setListener(Response.Listener {
                // SUCCESS
            })
            .setErrorListener(Response.ErrorListener {
                // ERROR
            })
            .setTag("TAG")
            .build()
```

### - Create POST request with Volley param
```kotlin
RequestBuilder<String>()
            .setMethod(Request.Method.POST)
            .setBaseUrl("https://httpbin.org")
            .setRelativeUrl("/post")
            .setResponseParser(StringResponseParser())
            .setListener(Response.Listener {
                // SUCCESS
            })
            .setErrorListener(Response.ErrorListener {
                // ERROR
            })
            .addParam("key1", "value1")
            .addParam("key2", "value2")
            .setTag("TAG")
            .build()
```

### - Create POST request with OkHttp3 RequestBody
```kotlin
RequestBuilder<String>()
            .setMethod(Request.Method.POST)
            .setBaseUrl("https://httpbin.org")
            .setRelativeUrl("/post")
            .setResponseParser(StringResponseParser())
            .setListener(Response.Listener {
                // SUCCESS
            })
            .setErrorListener(Response.ErrorListener {
                // ERROR
            })
            .setRequestBody(
                FormBody.Builder()
                    .add("key", "value")
                    .build()
            )
            .setTag("TAG")
            .build()
```

### - Execute Request
```kotlin
RequestBuilder<String>()
            .setMethod(Request.Method.GET)
            .setBaseUrl("https://httpbin.org")
            .setRelativeUrl("/post")
            .setResponseParser(StringResponseParser())
            .setListener(Response.Listener {
                // SUCCESS
            })
            .setErrorListener(Response.ErrorListener {
                // ERROR
            })
            .setTag("TAG")
            .build()
            .execute(requestQueue)
```

### - Execute Blocking Request
```kotlin
RequestBuilder<String>()
            .setMethod(Request.Method.GET)
            .setBaseUrl("https://httpbin.org")
            .setRelativeUrl("/post")
            .setResponseParser(StringResponseParser())
            .setTag("TAG")
            .build()
            .executeBlocking(requestQueue)
            .get()
```

### - Rx
```kotlin
RequestBuilder<String>()
            .setMethod(Request.Method.GET)
            .setBaseUrl("https://httpbin.org")
            .setRelativeUrl("/post")
            .setResponseParser(StringResponseParser())
            .setTag("TAG")
            .build()
            .single(requestQueue)
            .subscribeOn(Schedulers.io())
            .observeOn(AndroidSchedulers.mainThread())
            .subscribe()
            .apply { disposable.add(this) }
```

### - Coroutine
```kotlin
RequestBuilder<String>()
            .setMethod(Request.Method.GET)
            .setBaseUrl("https://httpbin.org")
            .setRelativeUrl("/post")
            .setResponseParser(StringResponseParser())
            .setTag("TAG")
            .build()
            .coroutine(requestQueue)
```

## RequestFuture.get() on main thread

RequestFuture.doGet() calls wait(0) on current thread. 

```java
// com.android.volley.toolbox.RequestFuture
private synchronized T doGet(Long timeoutMs)
        throws InterruptedException, ExecutionException, TimeoutException {
    if (mException != null) {
        throw new ExecutionException(mException);
    }

    if (mResultReceived) {
        return mResult;
    }

    if (timeoutMs == null) {
        while (!isDone()) {
            wait(0);
        }
    } else if (timeoutMs > 0) {
```

It will be awakened by RequestFuture.onResponse() or RequestFuture.onErrorResponse().

```java
// com.android.volley.toolbox.RequestFuture
@Override
public synchronized void onResponse(T response) {
    mResultReceived = true;
    mResult = response;
    notifyAll();
}

@Override
public synchronized void onErrorResponse(VolleyError error) {
    mException = error;
    notifyAll();
}
```

Those callbacks will be called on main thread if you use RequestQueue which was created by Volley.newRequestQueue() because Volley uses main looper Handler as default to send response to listener basically.

```java
// com.android.volley.RequestQueue
public RequestQueue(Cache cache, Network network, int threadPoolSize) {
    this(
        cache,
        network,
        threadPoolSize,
        new ExecutorDelivery(new Handler(Looper.getMainLooper())));
}
```

But wait(0) has been called already before that.

For this reason, if you don't create a new worker thread ExecuteDelivery,
RequestFuture.get() will block main thread forever.

So if you want to run RequestFuture.get() on main thread for any reason, please make ExecuteDelivery with new thread or use com.uoooo.volley.ext.toolbox.Volley.newRequestQueue().

- In short, thread that RequestQueue.get() was called and ExecutorDelivery's executor thread must be not identical.
