<a name="stat">
##获取资源信息

#### 描述
获取已存储的资源的相关metadata信息，而不会返回该资源本身。

#### HTTP请求

##### 语法

```
POST /stat/<EncodedEntryURI> HTTP/1.1
Host: rs.qiniu.com
```

##### 请求参数

该请求不支持任何请求参数。

##### 请求头

该请求必须指定以下这些请求头。

参数名称      | 说明                              | 必备性
:----------- | :------------------------------- | -------:
Authorization| QBox \<AccessToken\> <br> 关于AccessToken的详细格式，请参见 (http://docs.qiniu.com/xxx/xxx.html)            | 是
Content-Type | application/x-www-form-urlencoded| 是 

##### 请求正文

该请求无需指定任何正文内容。

#### HTTP响应

```
HTTP/1.1 200 OK
Content-Type: application/json
Cache-Control: no-store
Response Body: {
    fsize: <FileSize int>,
    hash: <FileETag string>,
    mimeType: <MimeType string>,
    putTime: <PutTime int64>  // 文件上传时候的七牛云存储服务器时间
}
```

#### 备注
该域名支持HTTP和HTTPS，建议始终采用HTTPS方式。
