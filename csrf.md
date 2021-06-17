项目名称:crsf
项目地址：https://github.com/gorilla/csrf
## 项目解决了什么问题
主要解决了cross-site request forgery (CSRF)跨站请求伪造攻击的问题
## 项目如何解决的
### http.Cookie 属性讲解
cookie 的 secure 属性，浏览器只能用https协议发送给服务器，http无法发送  
cookie 的 httponly属性，js脚本无法根据document.cookie 获取cookie的值  
cookie 的 samesite属性  Strict最为严格，完全禁止第三方 Cookie， Lax 允许 链接，预加载请求，GET 表单 发送cookie  
samesite设置为None必须同时设置secure属性，否则无效  
### csrf token生成
token的比较涉及到两部分 cookie中的token和前端提交上来的token  
原始的token： 32位的随机数  
cookie的token，通过securecookie进行转换并保存  
通过前端提交的token构造过程如下：  
生成一个32的随机数r，与token进行异或操作生成t，token' = base64(r+t) 这里的+表示拼接的操作  
所以当比较的时候，就从cookie中提取出正确的token然后与token'经过base64解码然后异或还原生成的token进行比较即可  
### 注意点
响应设置Vary：Cookie首部，当Cookie发生改变，一定要重新向源服务器发送请求而不是使用缓存  
HTTPS检查Referer是否是同源的，或者是受信任的源地址

