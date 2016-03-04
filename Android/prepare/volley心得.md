## Volley 心得  

卻达讲后，心得  



> 默认 Android2.3 及以上基于 HttpURLConnection，2.3 以下基于 HttpClient 实现  
> 系统在 Gingerbread 及之后(即 API Level >= 9)，采用基于 HttpURLConnection 的 HurlStack
> 如果小于 9，采用基于 HttpClient 的 HttpClientStack


**重要的请求队列 `mCacheQueue`， `mNetworkQueue`**    
**两个集合： `mWaitingRequests`, `mCurrentRequests`, 里面的集合是`Request`**  


* Volley.start();  

* Volley 对外暴露的 API，通过 newRequestQueue(…) 函数新建并启动一个请求队列RequestQueue  
	并返回当前requestQueue.

* Request: 表示请求的抽象类，`StringRequest`, `JsonRequest`,`ImageRequest`, 是子类。  
	具体是因为传入的参数是什么类型  ，即某种类型的请求  

* RequestQueue:  
	请求队列 调用mQueue.add(Request)，将一个request请求队列放入队列中，并排队，  
	将在工作线程中，划分为`mNetworkQueue`或`mCacheQueue`, 网络队列和缓存队列，  
	将response返回主线程  

	包含`CacheDispatcher` (用于处理走缓存请求的调度线程), `NetworkDispatcher数组` (用于处理走网络请求的调度线程)，  
	一个ResponseDelivery(返回结果分发接口).  
	通过`start()`,会启动`CacheDispatcher`和`NetworkDispatcher`

	* start() :   
		stop;//先将所有的调度线程都停止  
		重新创建并启动  
		将`mNetworkQueue`和`mCacheQueue`, 传入到`dispatcher`  
		cacheDispatcher创建一个就够了，networkDispatcher创建了多个
        network花费时间比较长，需要开多个线程来工作(默认会有四个线程为`network`中的`request`调度)  
		

* CacheDispatcher:  

	一个线程，用于调度处理走缓存的请求.  
	当`run()`时，会不断从缓存请求队列中取请求处理，  
	此时若队列为空则等待(PriorityBlockingQueue 决定的)。  
	因为在`run()`方法里面，  
	
	 	while(true) {
		...
	 	} 

	请求处理结束则将结果传递给ResponseDelivery去执行后续处理  

	当结果未缓存过、缓存失效或缓存需要刷新的情况下，该请求都需要重新进入NetworkDispatcher去调度处理  

* NetworkDispatcher:  

	一个线程，用于调度处理走网络的请求. 		
	启动后会不断从网络请求队列中取请求处理，队列为空则等待，  
	请求处理结束则将结果传递给ResponseDelivery去执行后续处理，  
	并判断结果是否要进行缓存

* ResponseDelivery ：

	上面都提过了`ResponseDelivery`, 它是返回结果的分发接口。  
	`ExecutorDelivery`实现了这个借口，实现了`postResponse(...)`同名不同参的三个方法  
	实现了`Runnable`  
	
* HttpStack（接口）:  

	处理Http请求，返回请求结果。  
	有基于 `HttpURLConnection` 的`HurlStack`和 基于 Apache `HttpClient` 的`HttpClientStack`。  
	`HttpClientStack` 和 `HurlStack` 均实现了接口 `HttpStack`  

* Network:  

	调用 `HttpStack` 处理请求，并将结果转换为可被`ResponseDelivery` 处理的`NetworkResponse`  
* Cache：  

	缓存请求结果，`Volley`默认使用的是基于`sdcard`的`DiskBasedCache`.  
	NetworkDispatcher得到请求结果后判断是否需要存储在 Cache，CacheDispatcher会从 Cache 中取缓存结果。  


* HttpStack.perforRequest(...)  
	return the HttpResponse, 返回网络请求的返回值  


> Done  
> chenzhao@hustunique.com  

 
