### HTTP请求线上Cases

#### Apache HttpClient
##### NoHttpResponseException: The target server failed to respond
* [线上问题分析：The target server failed to respond（目标服务器返回失败）](http://blog.csdn.net/ado1986/article/details/48268507)
 * 客户端使用了HttpClient的连接池机制，因此TCP连接会被重复使用，并且由于默认使用的是HTTP 1.1协议，Keep-Alive会被置上，因此不会主动关闭连接。如果客户端下一次请求正好使用了这个CLOSE_WAIT状态的TCP连接，就会出现服务器返回空应答的情况，从而抛出该问题。
 * HTTP协议是基于TCP连接的，如果是重用已有的TCP连接，则TCP三次握手不会发生。抛出该问题，是HttpClient在解析头信息时，发现没有数据。根本原因是因为没有返回HTTP数据包，而是返回了TCP数据包。

##### ProtocolException: The server failed to respond with a valid HTTP response
* [HttpClient问题：The server failed to respond with a valid HTTP response](http://blog.csdn.net/cctt_1/article/details/9021543)
 * 根本原因是：使用同一个HttpClient长连接/保持连接，然后又使用这个httpClient进行其他网络请求。

