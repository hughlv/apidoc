<a name="form-upload"></a>
## 表单上传

表单上传功能适用于文件内容可以在一次HTTP请求即可传递完成的场景。该功能非常适合于在浏览器中使用HTML表单上传资源，或者在不需要处理复杂情况的客户端开发中使用。

开发者只要组装一个符合**HTML文件上传表单**规范（参见[RFC1867](http://www.ietf.org/rfc/rfc1867.txt)）的HTTP请求，并以POST方式向域名`up.qiniu.com`发起这个请求，即可将指定文件上传到服务端。业务逻辑非常简单明了。

我们可以用如下的HTML表单来描述表单上传的基本用法：

```
<form method="post" action="http://up.qiniu.com/"
 enctype="multipart/form-data">
  <input name="key" type="hidden" value="<resource key>">
  <input name="token" type="hidden" value="<token>">
  <input name="file" type="file" />
</form>
```

该请求体中包含的`token`字段必须是一个符合相应规格的[上传凭证]()。

上传过程在一个HTTP请求和响应中完成，因此该过程将阻塞直到文件传输成功完成或失败为止。如果文件较大，或者网络环境较差，可能会导致HTTP连接超时而上传失败。若发生这种情况，开发者需要考虑换用更安全但也相对复杂的[切片上传]()功能。

提交以上这个HTML表单而生成的HTTP请求内容大致如下所示：

```
POST http://up.qiniu.com/
Content-Type: multipart/form-data; boundary=<Boundary>
--<Boundary>

Content-Disposition: form-data; name="key"
<FileID>
--<Boundary>

Content-Disposition: form-data; name="token"
<UploadToken>
--<Boundary>

Content-Disposition: form-data; name="file"; filename="<FileName>"
Content-Type: <MimeType>
<FileContent>
--<Boundary>--
```

在非网页开发的场景中，开发者完全可以自行组装这个HTML表单请求。考虑到各个平台上的网络库都已经对HTML文件上传表单有非常完整的支持，组装这个请求的过程将会非常轻松。

另外，表单上传过程还可以指定一系列的参数，以控制服务器在文件上传完成后的后续动作。我们将在[上传后续动作]()中详细描述各种参数的用法和作用。
