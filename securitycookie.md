项目名称：securecookie  
项目地址: https://github.com/gorilla/securecookie
### 基本介绍
项目主要介绍如何对cookie的值进行加密的一个具体实现  
字符说明：  
name 表示cookie的name  
value表示cookie的实际值  
b表示value经过转换后的值  
### 过程分析
1.第一步序列化 ,以json序列化实现过程为例
初始cookie(name string, value interface{})  
经过第一步b := e.Serialize(value)
```go
func (e JSONEncoder) Serialize(src interface{}) ([]byte, error) {
	buf := new(bytes.Buffer)
	enc := json.NewEncoder(buf)
	if err := enc.Encode(src); err != nil {
		return nil, cookieError{cause: err, typ: usageError}
	}
	return buf.Bytes(), nil
}
```
2.第二步 加密（可选）   
定义了blockKey之后就会进行这一步的操作  
使用aes.NewCipher函数生成cipher.Block，然后使用以下函数进行加密   
主要调用了  b = encrypt(block, b)
```go
//产生随机数的函数
func GenerateRandomKey(length int) []byte {
	k := make([]byte, length)
	if _, err := io.ReadFull(rand.Reader, k); err != nil {
		return nil
	}
	return k
}
//加密的函数
func encrypt(block cipher.Block, value []byte) ([]byte, error) {
	//这里通过随机数生成iv值
    iv := GenerateRandomKey(block.BlockSize())
	if iv == nil {
		return nil, errGeneratingIV
	}
	//  NewCTR 返回 a Stream which encrypts/decrypts using the given Block in
	// counter mode.
	stream := cipher.NewCTR(block, iv)
	stream.XORKeyStream(value, value)
	// Return iv + ciphertext.
	return append(iv, value...), nil
}
```
第三步： 创建	MAC 格式为 "name|date|value"
经过第二步加密的包含一些特殊字符，需要经过Base64编码方便后续处理  
Base64实现如下：
```go
// 使用base64编码
func encode(value []byte) []byte {
	encoded := make([]byte, base64.URLEncoding.EncodedLen(len(value)))
	base64.URLEncoding.Encode(encoded, value)
	return encoded
}
```
生成MAC值的过程如下：  
MAC主要用来保证cookie没有被修改过
```go
//这里先调用上面实现的Base64编码下
b = encode(b)
//封装成 "name|date|value" 其中name 就是cookie的key值 timestamp就是档期那时间戳time.Now().UTC().Unix()
b = []byte(fmt.Sprintf("%s|%d|%s|", name, s.timestamp(), b))
//这里创建主要利用hmac.New做一个hash的操作
//hash.New returns a new HMAC hash using the given hash.Hash type and key
mac := createMac(hmac.New(s.hashFunc, s.hashKey), b[:len(b)-1])
// 加上mac,删除 name.
b = append(b, mac...)[len(name)+1:]


//补充下createMac的具体实现
func createMac(h hash.Hash, value []byte) []byte {
	h.Write(value)
	return h.Sum(nil)
}
```
第四步： base64编码
```go
    b = encode(b)
```
第五步：检查长度
```go
     //超过最大长度就报错
     len(b) > s.maxLength 
```

反向过程说明：  
对照正向过程反向操作  
第一步：检查长度
```go
     //超过最大长度就报错
     len(b) > s.maxLength 
```
第二步：Base64解码
```go
    b = decode(b)
```
decode函数实现Base64解码实现
```go
func decode(value []byte) ([]byte, error) {
	decoded := make([]byte, base64.URLEncoding.DecodedLen(len(value)))
	b, err := base64.URLEncoding.Decode(decoded, value)
	if err != nil {
		return nil, cookieError{cause: err, typ: decodeError, msg: "base64 decode failed"}
	}
	return decoded[:b], nil
}
```
第三步：匹配MAC值  
此时cookie值格式是date|value|mac  
左侧添加key 然后计算新的mac值，然后与原来的mac匹配  
利用createMac函数,重新生成MAC值
cookie值中的date可用来判断过期时间
第四步：解密操作  
//先base64 转换成原来的字符，保持和原来完全相反的操作  
b  = decode(b)  
//然后通过decrypt解密  
b = decrypt(block, b)  
```go
func decrypt(block cipher.Block, value []byte) ([]byte, error) {
	size := block.BlockSize()
	if len(value) > size {
		// Extract iv.
		iv := value[:size]
		// Extract ciphertext.
		value = value[size:]
		// Decrypt it.
		stream := cipher.NewCTR(block, iv)
		stream.XORKeyStream(value, value)
		return value, nil
	}
	return nil, errDecryptionFailed
}
```
第五步：反序列化操作
value = e.Deserialize(b)
```go
// Deserialize decodes a value using encoding/json.
func (e JSONEncoder) Deserialize(src []byte, dst interface{}) error {
	dec := json.NewDecoder(bytes.NewReader(src))
	if err := dec.Decode(dst); err != nil {
		return cookieError{cause: err, typ: decodeError}
	}
	return nil
}
```

