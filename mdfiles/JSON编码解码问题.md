JSON执行编码过程中会对数字类型统一编码为number 类型,而在解码时会解码为flot64,因此在某些情况下,解码后数据会产生预期之外的错误,例如接口类型断言

因此,在go语言中,对一些场景下的编码解码工作可以使用gob序列化

编码过程

```
buf:=new(bytes.Buffer)
enc:=gob.NewEncoder(buf)//编码器
err:=enc.Encode(data)
b:=buf.Bytes() //序列化后的数据
```

解码过程

```
b:=buf.Bytes() //序列化后的数据
dec:=gob.NewDecoder(bytes.NewBuffer(b))
err:=dec.Decode(&data)//数据解码到data
```

