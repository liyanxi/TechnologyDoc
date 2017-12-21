## Retrofit
> `Retrofit官方文档翻译（原文链接：http://square.github.io/retrofit/）`
> 本文翻译纯属学习笔记记录，如有涉及到版权问题请邮件（itingchunyu@163.com）我，我会立即撤销展示，谢谢！

### <i>介绍</i>
`Retrofit ` 改造你的HTTP API变成一个Java接口。

```
public interface GitHubService {
	@Get("users/{user}/repos")
	Call<List<Repo>> listRepos(@Path("user") String user)
}
``` 
`Retrofit` 类生成一个 `GitHubService` 接口的实现。

```
Retrofit retrofit = new Retrofit.Builder()
	.baseUrl("https://api.github.com/") //可动态配置
	.build();
//生成指定接口的实现	
GitHubService service = retrofit.create(GitHubService.class);	
```
由`GitHubService`创建的每个 `Call` 可以同步或异步HTTP请求到远程网络服务器。
```
Call<List<Repo>> repos = service.listRepos("octocat");
```

使用注解去描述HTTP请求方式：

* URL中参数支持根据传入的查询参数动态调用替换
* 对象与请求实体的转换(如：JSON,protocol buffers)
* 多请求实体和文件上传支持

### API 声明
接口方法通过注解的方式及其参数指示如何处理一个请求

### 请求方式
每个方法必须拥有一个 `HTTP` 注解，去告知请求方式和相关调用Api。这里有五种构造方式：GET, POST, PUT, DELETE, and HEAD.相关资源的URL被指定在注释中。

```
@GET("users/list")
```

你也可以在URL中指定查询参数。

```
@GET("users/list?sort=desc")
```

### URL相关操作
在方法中使用替换块和参数可以动态更新一个请求的URL。一个替换块是一个字母数字字符串包围``{``和``}``。相应的参数必须与``@path``注释使用相同的字符串标记。

```
@GET("group/{id}/users")
Call<List<User>> groupList(@Path("id") int groupId);
```
查询参数也可以被添加。

```
@GET("group/{id}/users")
Call<List<User>> groupList(@Path("id") int groupId, @Query("sort") String sort);
```
对于复杂的查询参数，参数可以存放 ``Map`` 集合中。

```
@GET("group/{id}/users")
Call<List<User>> groupList(@Path("id") int groupId, @QueryMap Map<String, String> options);
```
### 请求 ``BODY``
使用``@Body``注释一个对象，这个对象可以作为HTTP请求实体传输。

```
@POST("users/new")
Call<User> createUser(@Body User user);
```
通过``Retrofit``实例初始化指定转换器，如此这个被注释的对象就可以被转换。如果初始化未指定，默认的``RequestBody``被使用。

### 表单编码和多部分提交
请求方法也可以被声明发送表单编码和多部分数据方式提交。

当 ``@FormUrlEncoded`` 注释出现在方法上，意味着请求以表单变编码方式提交。因此每个``key-value``键值对参数必须被``@Field``标记注释，包含名称和对应的值。

```
@FormUrlEncoded
@POST("user/edit")
Call<User> update(@Field("first_name") String first, @Field("last_name") String last);
```
Multipart requests are used when @Multipart is present on the method. Parts are declared using the @Part annotation.

```
@Multipart
@PUT("user/photo")
Call<User> updateUser(@Part("photo") RequestBody photo, @Part("description") RequestBody description);
```
### 请求Headers参数设置
你可以根据需要在一个方法上使用``@Headers``设置静态的头参数配置。

```
@Headers("Cache-Control: max-age=640000")
@GET("widget/list")
Call<List<Widget>> widgetList();
```
```
@Headers({
	"Accept: application/vnd.github.v3.full+json",
	"User-Agent: Retrofit-Sample-App"
})
@GET("users/{username}")
Call<User> getUser(@Path("username") String username);
```
注意,``Headers``不会互相覆盖。具有相同名称的所有头文件都包括在请求。一个请求的``Headers``可以动态更新。通过``@Header``注释相关参数传入。如果传入的值为空，这个header将会被忽略。否则``toString``方法将会被调用并且结果将被使用。

```
@GET("user")
Call<User> getUser(@Header("Authorization") String authorization);
```
``Headers``可以使用[OkHttp interceptor][0]为每个请求统一指定headers。

### 同步Vs异步
``Call`` 事例能被同步或异步执行。每个事例仅仅能被使用一次，但是调用``clone()``方法可以创建一个副本可以被重新使用。
在Android里面，回调执行在主线程。在Java虚拟机中，回调将会发生在与发起HTTP请求同一个线程中。

### Retrofit配置
``Retrofit``是一种能将你的API接口转化为可调用的对象的类。默认情况下,``Retrofit``会给你默认配置项,但它允许你自己定制。

### 转换器
默认情况下，``Retrofit``可以反序列化``HTTP bodyies``为OkHttp的``ResponseBody``类型，并且它只接受``RequestBody``实体为``@Body``注释的类型。
转换器也可以添加其它类型支持。六个模块适应现下主流的序列化库为你提供方便。

* [Gson][1]：com.squareup.retrofit2:converter-gson
* [Jackson][2]：com.squareup.retrofit2:converter-jackson
* [Moshi][3]：com.squareup.retrofit2:converter-moshi
* [Protobuf][4]：com.squareup.retrofit2:converter-protobuf
* [Wire][5]：com.squareup.retrofit2:converter-wire
* [Simple XML][6]：com.squareup.retrofit2:converter-simplexml
* [Scalars](primitives,boxed,and String)：com.squareup.retrofit2:converter-scalars

下面是一个使用GsonConverterFactory类生成GitHubService接口的实现的示例，该接口使用Gson进行反序列化。

```
Retrofit retrofit = new Retrofit.Builder()
	.baseUrl("https://api.github.com")
	.addConverterFactory(GsonCoverterFactory.create())
	.build();
GitHubService service = retrofit.create(GitHubService.class);
```
### 自定义转换器
如果您需要使用Retrofit不支持的内容格式（例如YAML，txt，自定义格式）的API进行通信，或者您希望使用不同的库来实现现有的格式，则可以轻松地创建你自己的转换器。 创建一个扩展了Converter.Factory类的类，并在构建适配器时传递一个实例。

### Download
[V2.3.0 JAR][7]

`Retrofit`的源代码、事例你可以在[GitHub][8]上查阅。

### Maven

```
<dependency>
  <groupId>com.squareup.retrofit2</groupId>
  <artifactId>retrofit</artifactId>
  <version>2.3.0</version>
</dependency>
```

### Gradle依赖
```
compile 'com.squareup.retrofit2:retrofit:2.3.0'
```
`Retrofit`要求最小Java7或Android 2.3.

### 混淆机制
如果你使用在你的项目中使用`ProGuard`，你应该在你的混淆文件配置中添加下面对应混淆代码：

```
# Platform calls Class.forName on types which do not exist on Android to determine platform.
-dontnote retrofit2.Platform
# Platform used when running on Java 8 VMs. Will not be used at runtime.
-dontwarn retrofit2.Platform$Java8
# Retain generic type information for use by reflection by converters and adapters.
-keepattributes Signature
# Retain declared checked exceptions for use by a Proxy instance.
-keepattributes Exceptions
```

因`Retrofit`的底层使用`Okio`，所以你或许也需要了解[ProGuard rules][9]混淆规则。

------------------
### <i>License</i>

```
Copyright 2013 Square, Inc.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

   http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```

[0]:https://github.com/square/okhttp/wiki/Interceptors
[1]:https://github.com/google/gson
[2]:http://wiki.fasterxml.com/JacksonHome
[3]:https://github.com/square/moshi/
[4]:https://developers.google.com/protocol-buffers/
[5]:https://github.com/square/wire
[6]:http://simple.sourceforge.net/
[7]:http://repo1.maven.org/maven2/com/squareup/retrofit2/retrofit/2.3.0/retrofit-2.3.0.jar
[8]:https://github.com/
[9]:https://github.com/square/okio#proguard




