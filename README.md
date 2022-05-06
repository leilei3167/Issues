# Issues

## 阅读IO包

io.go:659:

```go
func ReadAll(r Reader) ([]byte, error) {
	b := make([]byte, 0, 512)
	for {
		if len(b) == cap(b) {
			// Add more capacity (let append pick how much).
			b = append(b, 0)[:len(b)] //这种用法是什么意思?
		}
		n, err := r.Read(b[len(b):cap(b)])
		b = b[:len(b)+n]
		if err != nil {
			if err == EOF {
				err = nil
			}
			return b, err
		}
	}
}
```

- 解析:`func append(slice []Type, elems ...Type) []Type` append后可直接跟切片截取操作,这一步的意思是,随便加入一个元素触发扩容,  
但是只截取[:len(b)]个元素,即舍弃刚加入的元素,但是cap已经是扩容后的  

- Read只会读取len(p)个字节,并返回读取的字节数n,此处`b[len(b):cap(b)]`刚好就是当前缓冲区空闲的空间,向其中写入,`b = b[:len(b)+n]`  
将为写入数据的空闲部分舍去,读取到EOF后退出(EOF在此处不被视为错误)
