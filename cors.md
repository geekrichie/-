## 简要介绍
https://github.com/rs/cors  
看完了cors包的具体实现，  
里面主要通过设置了一些HTTP的HEADER来达到管理跨域请求的目的，  
最大的收获在于学会了很多HTTP HEADER的用法  
项目主要实现到了一下的HTTP的用法，详细可参考https://developer.mozilla.org/zh-CN/docs/Web/HTTP  
一般来说POST和CORS都会有Origin首部，  
但是任何时候都应该设置Vary: Origin  
避免代理服务器做出错误的相应，此时代理服务器会根据Origin来选择是否使用缓存
## 跨域相关的HEADER功能说明
|  字段名称   | 字段说明  |
|  ----  | ----  |
|  Origin     |  \<scheme> "://" \<host> [ ":" \<port> ] <br>包含协议主机端口号   |
|  X-Requested-With| 根据这个的值判断是ajax请求还是正常请求|
|  Vary | 告诉代理服务器或者浏览器哪些头部被用来判断缓存是否可用|
|  Access-Control-Request-Method   |在预检请求用于通知服务器在真正的请求中会采用哪种  HTTP 方法 |
|  Access-Control-Request-Headers |用于通知服务器在真正的请求中会采用哪些请求头|
|  Access-Control-Max-Age|（预检请求）的返回结果可以缓存的时间|
|  Access-Control-Allow-Origin |响应头指定了该响应的资源是否被允许与给定的origin共享。|
|  Access-Control-Allow-Methods| 在对（预检请求）的应答中明确了客户端所要访问的资源允许使用的方法或方法列表。|
|  Access-Control-Allow-Headers| 将会在正式请求的 Access-Control-Request-Headers 字段中出现的首部信息。|
|  Access-Control-Allow-Credentials| 响应头表示是否可以将对请求的响应暴露给前端JS代码|
|  Access-Control-Expose-Headers| 列出了哪些首部可以作为响应的一部分暴露给外部。正常只提供这些<br>Cache-Control<br>Content-Language<br>Content-Length<br>Content-Type<br>Expires<br>Last-Modified<br>Pragma|
## 项目
CORS分为两步，第一步为预检请求，OPTIONS会带上

|  字段名称   | 字段说明  |
|  ----  | ----  |
|  Access-Control-Request-Method   |在预检请求用于通知服务器在真正的请求中会采用哪种  HTTP 方法 |
|  Access-Control-Request-Headers |用于通知服务器在真正的请求中会采用哪些请求头|
响应中包含如下响应头

|  字段名称   | 字段说明  |
|  ----  | ----  |
|  Access-Control-Max-Age|（预检请求）的返回结果可以缓存的时间|
|  Access-Control-Allow-Origin |响应头指定了该响应的资源是否被允许与给定的origin共享。|
|  Access-Control-Allow-Methods| 在对（预检请求）的应答中明确了客户端所要访问的资源允许使用的方法或方法列表。|
|  Access-Control-Allow-Headers| 将会在正式请求的 Access-Control-Request-Headers 字段中出现的首部信息。|
|  Access-Control-Allow-Credentials| 响应头表示是否可以将对请求的响应暴露给前端JS代码|
|  Access-Control-Expose-Headers| 列出了哪些首部可以作为响应的一部分暴露给外部。正常只提供这些<br>Cache-Control<br>Content-Language<br>Content-Length<br>Content-Type<br>Expires<br>Last-Modified<br>Pragma|  
正式请求只返回

|  字段名称   | 字段说明  |
|  ----  | ----  |
|  Access-Control-Allow-Origin |响应头指定了该响应的资源是否被允许与给定的origin共享。|
|  Access-Control-Allow-Credentials| 响应头表示是否可以将对请求的响应暴露给前端JS代码|
|  Access-Control-Expose-Headers| 列出了哪些首部可以作为响应的一部分暴露给外部。正常只提供这些<br>Cache-Control<br>Content-Language<br>Content-Length<br>Content-Type<br>Expires<br>Last-Modified<br>Pragma|  
