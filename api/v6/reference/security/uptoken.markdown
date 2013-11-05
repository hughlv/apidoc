## 上传凭证（UploadToken）

上传凭证是七牛云存储用于验证上传请求合法性的机制。用户通过上传凭证授权客户端，使其具备访问指定资源的能力。

<a name="putpolicy"></a>
### 上传策略（PutPolicy）

上传策略是资源上传时的一组配置设定。通过这组配置信息，七牛云存储可以了解用户上传的需求：它将上传什么资源，上传到哪个空间，是否需要回调，还是使用重定向，是否需要设置反馈信息的内容，以及请求的失效时间等等。

上传策略同时还参与请求验证。实际上，上传凭证就是上传策略的加密结果。通过对PutPolicy的加密，可以确保用户对某个资源的上传请求是完全受到验证的。

上传策略的具体构成如下：

```
{
    scope: <Bucket string>,
    deadline: <UnixTimestamp int64>,
    endUser: <EndUserId string>,
    returnUrl: <RedirectURL string>,
    returnBody: <ResponseBodyForAppClient string>,
    callbackBody: <RequestBodyForAppServer string>
    callbackUrl: <RequestUrlForAppServer string>,
    asyncOps: <asyncProcessCmds string>
}
```

**参数详解** ：

 字段名       | 必须 | 说明
--------------|------|-----------------------------------------------------------------------
 scope        | 是   | 用于指定文件所上传的目标资源空间（Bucket）和资源名（Key）。格式为：\<bucket name\>\[:\<key\>\]。若只指定Bucket名，表示文件上传至该Bucket。若同时指定了Bucket和Key（\<bucket name\>:\<key\>），表示上传文件限制在指定的Key上。两种形式的差别在于，前者是“新增”操作：如果所上传文件的Key在Bucket中已存在，上传操作将失败。而后者则是“新增或覆盖”操作：如果Key在Bucket中已经存在，将会被覆盖；如不存在，则将文件新增至Bucket中。<p>**注意：资源名必须采用utf8编码，非utf8编码的资源名在访问七牛云存储将会反馈错误。**
 deadline     | 是   | 定义上传请求的失效时间，[Unix时间戳](http://en.wikipedia.org/wiki/Unix_time)，单位为秒。
 endUser      | 否   | 给上传的文件添加唯一属主标识，特殊场景下非常有用，比如根据App-Client标识给图片或视频打水印
 returnUrl    | 否   | 设置用于浏览器端文件上传成功后，浏览器执行301跳转的URL，一般为`HTML Form`上传时使用。文件上传成功后会跳转到`returnUrl?query_string`, `query_string`会包含 `returnBody` 内容。
 returnBody   | 否   | 文件上传成功后，自定义从七牛云存储最终返回給终端 App-Client 的数据。支持 [魔法变量](#MagicVariables)和[自定义变量](#xVariables)。
 callbackBody | 否   | 文件上传成功后，七牛云存储向 App-Server 发送POST请求的数据。支持 [魔法变量](#MagicVariables) 和 [自定义变量](#xVariables)。
 callbackUrl  | 否   | 文件上传成功后，七牛云存储向 App-Server 发送POST请求的URL，必须是公网上可以正常进行POST请求并能响应 HTTP Status 200 OK 的有效 URL 
 asyncOps     | 否   | 指定文件（图片/音频/视频）上传成功后异步地执行指定的预转操作。每个预转指令是一个API规格字符串，多个预转指令可以使用分号`;`隔开。具体云处理指令请参考[fop](http://docs.qiniu.com/api/v6/gen-use.html#fop)

**注意**

- `callbackUrl` 与 `returnUrl` 不可同时指定，两者只可选其一。
- `callbackBody` 与 `returnBody` 不可同时指定，两者只可选其一。

### 凭证算法

上传凭证的算法如下：

1. 构造[上传策略](#put-policy)。用户根据业务需求，确定上传策略的要素，构造出具体的上传策略。比如，有用户需要向空间 `my-bucket` 上传一个名为 `sunflower.jpg` 的图片，有效期是到 `2015-12-31 00:00:00`，并且希望得到图片的名称、大小、宽、高和校验值。那么相应的上传策略的字段分别为：

    ```
    scope = "my-bucket:sunflower.jpg"
    deadline = 1451491200
    returnBody = '{
      "name": $(fname),
      "size": $(fsize),
      "w": $(imageInfo.width),
      "h": $(imageInfo.height),
      "hash": $(etag),
    }'
    ```

1. 将上传策略序列化成为json格式。用户可以使用各种语言的json库，也可以手工地拼接字符串。序列化后，可以得到：

    ```
    put_policy = '{"scope":"my-bucket:sunflower.jpg","deadline":1451491200,"returnUrl":"{\"name\": $(fname),\"size\": $(fsize),\"w\": $(imageInfo.width),\"h\": $(imageInfo.height),\"hash\": $(etag),}"}'
    ```

1. 对json序列化后的上传策略进行[URL安全的Base64编码](http://docs.qiniu.com/api/v6/terminology.html#URLSafeBase64)：

    ```
    encoded = urlsafe_base64_encode(put_policy)
    ```

    得到

    ```
    "eyJzY29wZSI6Im15LWJ1Y2tldDpzdW5mbG93ZXIuanBnIiwiZGVhZGxpbmUiOjE0NTE0OTEyMDAsInJldHVyblVybCI6IntcIm5hbWVcIjogJChmbmFtZSksXCJzaXplXCI6ICQoZnNpemUpLFwid1wiOiAkKGltYWdlSW5mby53aWR0aCksXCJoXCI6ICQoaW1hZ2VJbmZvLmhlaWdodCksXCJoYXNoXCI6ICQoZXRhZyksfSJ9"
    ```

1. 用SecretKey对编码后的上传策略进行HMAC-SHA1加密，并且做URL安全的Base64编码：

    ```
    signature = hmac_sha1(SecretKey, encoded)
    encode_signed = urlsafe_base64_encode(signature)
    ```

    假设用户的 `SecretKey="Yx0hNBifQ5V5SqLUkzPkjyy0pbYJpav9CH1QzkG0"` 加密后的结果是：

    ```
    "5Cr3Nrw0qkyYKfQicd_ejAdIrfs="
    ```

1. 最后，将 `AccessKey`、`encode_signed` 和 `encoded` 用 “:” 连接起来：

    ```
    upload_token = AccessKey + ":" + encode_signed + ":" + encoded
    ```
    
    假设用户的 `AccessKey="j6XaEDm5DwWvn0H9TTJs9MugjunHK8Cwo3luCglo"` 。最后得到的上传凭证为：

    ```
    j6XaEDm5DwWvn0H9TTJs9MugjunHK8Cwo3luCglo:5Cr3Nrw0qkyYKfQicd_ejAdIrfs=:eyJzY29wZSI6Im15LWJ1Y2tldDpzdW5mbG93ZXIuanBnIiwiZGVhZGxpbmUiOjE0NTE0OTEyMDAsInJldHVyblVybCI6IntcIm5hbWVcIjogJChmbmFtZSksXCJzaXplXCI6ICQoZnNpemUpLFwid1wiOiAkKGltYWdlSW5mby53aWR0aCksXCJoXCI6ICQoaW1hZ2VJbmZvLmhlaWdodCksXCJoYXNoXCI6ICQoZXRhZyksfSJ9
    ```

### 代码示例

```
// TODO: 代码示例goes here.
```