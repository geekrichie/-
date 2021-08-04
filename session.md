# session 的实现方式
session 通常会被用于保存用户登录的状态信息  
以前总是好奇session的实现方式，所以看了这个包之后，也算弄懂了session的工作原理  
# 工作原理
创建session保存在服务端，以内存为例
包含一个 
```go
//单个用户的session，通过sid标记
type dataItem struct {
	sid       string
	expiredAt time.Time
	values    map[string]interface{}
}
//保存所有用户的session
type memoryStore struct {
	sync.RWMutex
	data   map[string]*dataItem
	list   *list.List
	ticker *time.Ticker
}
```
其中dataItem对应每个用户的session，通过一个sid来标记,  
包含一个过期时间和一个map用来存储键值对。  
当用户A登录的时候，生成一个dataItem， 其中sid 借助UUID算法  
生成，values保存一个键值对{ "islogin" ： true } 即可  
UUID生成算法
```go
func newUUID() string {
	var buf [16]byte
	io.ReadFull(rand.Reader, buf[:])
	buf[6] = (buf[6] & 0x0f) | 0x40
	buf[8] = (buf[8] & 0x3f) | 0x80

	dst := make([]byte, 36)
	hex.Encode(dst, buf[:4])
	dst[8] = '-'
	hex.Encode(dst[9:13], buf[4:6])
	dst[13] = '-'
	hex.Encode(dst[14:18], buf[6:8])
	dst[18] = '-'
	hex.Encode(dst[19:23], buf[8:10])
	dst[23] = '-'
	hex.Encode(dst[24:], buf[10:])

	return string(dst)
}
```

生成sid之后，需要保存到浏览器的cookie中或者http header里面  
这样下次用户提交请求的时候带上cookie，然后后端就可以通过sid  
取memoryStore里面去查找对应的dataItem，然后获取其中的键值对了  