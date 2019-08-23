---
layout: post
title: "Retrofit请求流程分析"
subtitle: "本文记录Retrofit请求流程"
author: "XYH"
header-img: ""
header-bg-css: "linear-gradient(to right, #24b94a, #38ef7d);"
tags:
  - Android
  - Retrofit  
---

>本文记录Retrofit一个请求的生成流程。

# 前言

早就想去深入分析的一个网络库，最近终于能静下心去分析它的请求流程，本文 [Retrofit](https://github.com/square/retrofit) 版本为`v2.4.0`。

本文追踪一个接口里的方法怎么变成`Call<T>`，这个清晰之后便可知[Retrofit](https://github.com/square/retrofit) 基本工作流程。

```java
class MainActivity : AppCompatActivity() {
    private val mRetrofit by lazy {
        Retrofit.Builder()
            .baseUrl("http://gank.io")
            .build()
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        btn_request.setOnClickListener {
            val api = mRetrofit.create(API::class.java)
            val mCall = api.getCategory()
            mCall.enqueue(object : Callback<ResponseBody> {
                override fun onFailure(call: Call<ResponseBody>, t: Throwable) {

                }

                override fun onResponse(call: Call<ResponseBody>, response: Response<ResponseBody>) {

                }
            })
        }
    }

    interface API {
        @GET("/api/xiandu/categories")
        fun getCategory(): Call<ResponseBody>
    }
}
```

测试请求使用 [gank.io](http://gank.io) 开放api。

从 `create` 方法入手。

## create

`Retrofit#create()`使用了动态代理，重点看一下InvocationHandler#invoke()的实现。

```java
public <T> T create(final Class<T> service) {
   //确保传入的对象是接口并且不能继承其他接口。
    Utils.validateServiceInterface(service);
    //是否在create时立即缓存传入对象声明的方法
    if (validateEagerly) {
      eagerlyValidateMethods(service);
    }
    return (T) Proxy.newProxyInstance(service.getClassLoader(), new Class<?>[] { service },
        new InvocationHandler() {
          private final Platform platform = Platform.get();

          @Override public Object invoke(Object proxy, Method method, @Nullable Object[] args)
              throws Throwable {
            // If the method is a method from Object then defer to normal invocation.
            if (method.getDeclaringClass() == Object.class) {
              return method.invoke(this, args);
            }
            if (platform.isDefaultMethod(method)) {
              return platform.invokeDefaultMethod(method, service, proxy, args);
            }
			//见下文
            ServiceMethod<Object, Object> serviceMethod =
                (ServiceMethod<Object, Object>) loadServiceMethod(method);
            OkHttpCall<Object> okHttpCall = new OkHttpCall<>(serviceMethod, args);
            return serviceMethod.adapt(okHttpCall);
          }
        });
  }
```


```java
  ServiceMethod<?, ?> loadServiceMethod(Method method) {
    //serviceMethodCache 是 Retrofit下声明的一个ConcurrentHashMap对象，用于缓存retrofit#create(service)中service的方法
    //ServiceMethod的职责就是将一个在service接口声明的方法转换成一个httpCall，见下文的ServiceMethod
    ServiceMethod<?, ?> result = serviceMethodCache.get(method);
    if (result != null) return result;

    synchronized (serviceMethodCache) {
   
      result = serviceMethodCache.get(method);
      if (result == null) {
        result = new ServiceMethod.Builder<>(this, method).build();
        serviceMethodCache.put(method, result);
      }
    }
    return result;
  }
```

### Platform

本文以Android为例。

```java

 static class Android extends Platform {
    @Override public Executor defaultCallbackExecutor() {
      return new MainThreadExecutor();
    }

    @Override CallAdapter.Factory defaultCallAdapterFactory(@Nullable Executor callbackExecutor) {
      if (callbackExecutor == null) throw new AssertionError();
      return new ExecutorCallAdapterFactory(callbackExecutor);
    }

    static class MainThreadExecutor implements Executor {
      private final Handler handler = new Handler(Looper.getMainLooper());

      @Override public void execute(Runnable r) {
        handler.post(r);
      }
    }
}
```

## ServiceMethod

**重点**：该类会解析出请求类型、请求参数、请求头等所有请求信息。

对应方法为`parseMethodAnnotation`、`parseHttpMethodAndPath`、`parseHeaders`、`parseParameterAnnotation`

请求类型、请求头、参数会在ServiceMethod#Builder#build()的时候全部解析。

回到Retrofit#create()中。

```java
  public <T> T create(final Class<T> service) {
    Utils.validateServiceInterface(service);
    if (validateEagerly) {
      eagerlyValidateMethods(service);
    }
    return (T) Proxy.newProxyInstance(service.getClassLoader(), new Class<?>[] { service },
        new InvocationHandler() {
          private final Platform platform = Platform.get();

          @Override public Object invoke(Object proxy, Method method, @Nullable Object[] args)
              throws Throwable {
            // If the method is a method from Object then defer to normal invocation.
            if (method.getDeclaringClass() == Object.class) {
              return method.invoke(this, args);
            }
            //Android平台默认false
            if (platform.isDefaultMethod(method)) {
              return platform.invokeDefaultMethod(method, service, proxy, args);
            }
            ServiceMethod<Object, Object> serviceMethod =
                (ServiceMethod<Object, Object>) loadServiceMethod(method);
            //包装的OkhttpCall
            OkHttpCall<Object> okHttpCall = new OkHttpCall<>(serviceMethod, args);
            //adapter返回一个Call，默认情况下为ExecutorCallAdapterFactory.get().adapt(call)
            return serviceMethod.adapt(okHttpCall);
          }
        });
  }
```

点开 `serviceMethod.adapt(okHttpCall);`

## serviceMethod.adapt(okHttpCall)

 ```java
   T adapt(Call<R> call) {
    return callAdapter.adapt(call);
  }
 ```
 
 callAdapter为何物？
 
 ```java
 public interface CallAdapter<R, T> {
 
  Type responseType();

  
  T adapt(Call<R> call);

 
    public abstract @Nullable CallAdapter<?, ?> get(Type returnType, Annotation[] annotations,
        Retrofit retrofit);

   
    protected static Type getParameterUpperBound(int index, ParameterizedType type) {
      return Utils.getParameterUpperBound(index, type);
    }

   
    protected static Class<?> getRawType(Type type) {
      return Utils.getRawType(type);
    }
  }
}

```
可以理解为`Call`的生成代理。

`ServiceMethod#callAdapter`从何而来？

位于 `ServiceMethod#Builder.build`。

```java
class ServiceMethod {
    xxx
     static final class Builder<T, R> {
     
        public ServiceMethod build() {
          callAdapter = createCallAdapter();
        }
        
        private CallAdapter<T, R> createCallAdapter() {
          Type returnType = method.getGenericReturnType();
          if (Utils.hasUnresolvableType(returnType)) {
            throw methodError(
                "Method return type must not include a type variable or wildcard: %s", returnType);
          }
          if (returnType == void.class) {
            throw methodError("Service methods cannot return void.");
          }
          Annotation[] annotations = method.getAnnotations();
          try {
            return (CallAdapter<T, R>) retrofit.callAdapter(returnType, annotations);
          } catch (RuntimeException e) { // Wide exception range because factories are user code.
            throw methodError(e, "Unable to create call adapter for %s", returnType);
          }
        }
    
        xxx
    }
    xxx
}

```

回调到`Retrofit#callAdapter(returnType,annotations)`

```java
class Retrofit{

    public CallAdapter<?, ?> callAdapter(Type returnType, Annotation[] annotations) {
        return nextCallAdapter(null, returnType, annotations);
    }
    
    public CallAdapter<?, ?> nextCallAdapter(@Nullable CallAdapter.Factory skipPast, Type returnType,
      Annotation[] annotations) {
        checkNotNull(returnType, "returnType == null");
        checkNotNull(annotations, "annotations == null");
    
        int start = callAdapterFactories.indexOf(skipPast) + 1;
        for (int i = start, count = callAdapterFactories.size(); i < count; i++) {
          //获取callAdapter
          CallAdapter<?, ?> adapter = callAdapterFactories.get(i).get(returnType, annotations, this);
          if (adapter != null) {
            return adapter;
          }
        }
    
        StringBuilder builder = new StringBuilder("Could not locate call adapter for ")
            .append(returnType)
            .append(".\n");
        if (skipPast != null) {
          builder.append("  Skipped:");
          for (int i = 0; i < start; i++) {
            builder.append("\n   * ").append(callAdapterFactories.get(i).getClass().getName());
          }
          builder.append('\n');
        }
        builder.append("  Tried:");
        for (int i = start, count = callAdapterFactories.size(); i < count; i++) {
          builder.append("\n   * ").append(callAdapterFactories.get(i).getClass().getName());
        }
        throw new IllegalArgumentException(builder.toString());
  }
}
```

继续看 `callAdapterFactories`在哪里赋值。

```java
public final class Retrofit {
  private final Map<Method, ServiceMethod<?, ?>> serviceMethodCache = new ConcurrentHashMap<>();

  final okhttp3.Call.Factory callFactory;
  final HttpUrl baseUrl;
  final List<Converter.Factory> converterFactories;
  final List<CallAdapter.Factory> callAdapterFactories;
  final @Nullable Executor callbackExecutor;
  final boolean validateEagerly;

  Retrofit(okhttp3.Call.Factory callFactory, HttpUrl baseUrl,
      List<Converter.Factory> converterFactories, List<CallAdapter.Factory> callAdapterFactories,
      @Nullable Executor callbackExecutor, boolean validateEagerly) {
    this.callFactory = callFactory;
    this.baseUrl = baseUrl;
    this.converterFactories = converterFactories; // Copy+unmodifiable at call site.
    //callAdapterFactories赋值
    this.callAdapterFactories = callAdapterFactories; // Copy+unmodifiable at call site.
    this.callbackExecutor = callbackExecutor;
    this.validateEagerly = validateEagerly;
  }
  
   public static final class Builder { 
       xxx
        public Retrofit build() {
      if (baseUrl == null) {
        throw new IllegalStateException("Base URL required.");
      }

      okhttp3.Call.Factory callFactory = this.callFactory;
      if (callFactory == null) {
        callFactory = new OkHttpClient();
      }
      //默认的callbackExecutor
      Executor callbackExecutor = this.callbackExecutor;
      if (callbackExecutor == null) {
        callbackExecutor = platform.defaultCallbackExecutor();
      }

      // callAdapterFactories赋值
      List<CallAdapter.Factory> callAdapterFactories = new ArrayList<>(this.callAdapterFactories);
      callAdapterFactories.add(platform.defaultCallAdapterFactory(callbackExecutor));

      List<Converter.Factory> converterFactories =
          new ArrayList<>(1 + this.converterFactories.size());

      converterFactories.add(new BuiltInConverters());
      converterFactories.addAll(this.converterFactories);

      return new Retrofit(callFactory, baseUrl, unmodifiableList(converterFactories),
          unmodifiableList(callAdapterFactories), callbackExecutor, validateEagerly);
    }
       xxx
   }
}
```

** Retrofit在构造时提供了默认的CallAdapterFactory来获取CallAdapter。**

查看 `platform.defaultCallAdapterFactory(callbackExecutor)`。

该方法后续会返回。`ExecutorCallAdapterFactory`。

所以Retrofit#create里面的invocationHandler#invoke最终调用的是`ExecutorCallAdapterFactory#get().adapt(Call<Object> call)`。

```java
final class ExecutorCallAdapterFactory extends CallAdapter.Factory {
  final Executor callbackExecutor;

  ExecutorCallAdapterFactory(Executor callbackExecutor) {
    this.callbackExecutor = callbackExecutor;
  }

  @Override
  public CallAdapter<?, ?> get(Type returnType, Annotation[] annotations, Retrofit retrofit) {
    if (getRawType(returnType) != Call.class) {
      return null;
    }
    final Type responseType = Utils.getCallResponseType(returnType);
    return new CallAdapter<Object, Call<?>>() {
      @Override public Type responseType() {
        return responseType;
      }

      @Override public Call<Object> adapt(Call<Object> call) {
        return new ExecutorCallbackCall<>(callbackExecutor, call);
      }
    };
  }

  static final class ExecutorCallbackCall<T> implements Call<T> {
    final Executor callbackExecutor;
    final Call<T> delegate;

    ExecutorCallbackCall(Executor callbackExecutor, Call<T> delegate) {
      this.callbackExecutor = callbackExecutor;
      this.delegate = delegate;
    }

    @Override public void enqueue(final Callback<T> callback) {
      checkNotNull(callback, "callback == null");

      delegate.enqueue(new Callback<T>() {
        @Override public void onResponse(Call<T> call, final Response<T> response) {
          callbackExecutor.execute(new Runnable() {
            @Override public void run() {
              if (delegate.isCanceled()) {
                // Emulate OkHttp's behavior of throwing/delivering an IOException on cancellation.
                callback.onFailure(ExecutorCallbackCall.this, new IOException("Canceled"));
              } else {
                callback.onResponse(ExecutorCallbackCall.this, response);
              }
            }
          });
        }

        @Override public void onFailure(Call<T> call, final Throwable t) {
          callbackExecutor.execute(new Runnable() {
            @Override public void run() {
              callback.onFailure(ExecutorCallbackCall.this, t);
            }
          });
        }
      });
    }

    @Override public boolean isExecuted() {
      return delegate.isExecuted();
    }

    @Override public Response<T> execute() throws IOException {
      return delegate.execute();
    }

    @Override public void cancel() {
      delegate.cancel();
    }

    @Override public boolean isCanceled() {
      return delegate.isCanceled();
    }

    @SuppressWarnings("CloneDoesntCallSuperClone") // Performing deep clone.
    @Override public Call<T> clone() {
      return new ExecutorCallbackCall<>(callbackExecutor, delegate.clone());
    }

    @Override public Request request() {
      return delegate.request();
    }
  }
}
```

至此，一个Call的生成大致流程如此。

那怎么配合RxJava返回Observeable的？

Retrofit提供了`addCallAdapterFactory`来让开发者自由干预Call的生成，至于原理跟上面一致了。
返回值的处理也大同小异。

## 总结

看完Retrofit的设计，让人觉得受益匪浅，知识的合理运用外加上设计模式的巧妙搭配能写出让人舒服的代码。



