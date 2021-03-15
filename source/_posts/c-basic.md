---
title: windows下面使用libcurl
date: 2021-03-15 20:53:59
tags:
---

1. 下载安装
下载 libcurl -- http://curl.haxx.se/dlwiz/

2. 设置windows环境变量
LIBCURL_ROOT -- libcurl 存放路径，比如 E:/opensdk/libcurl-7.19.3

3. 设置VS工程属性，添加$(LIBCURL_ROOT)/include, $(LIBCURL_ROOT)/lib, 动态链接添加curllib.lib，静态链接添加curllib_static.lib
静态链接需添加如下东西：
#pragma comment ( lib, "ws2_32.lib" )
#pragma comment ( lib, "wldap32.lib" )
CURL_STATICLIB

If you’re using libcurl as a win32 DLL, you MUST use the CURLOPT_WRITEFUNCTION if you set CURLOPT_
WRITEDATA - or you will experience crashes.

4. 使用libcurl操作HTTP协议必须考虑的问题：
4.1 操作是同步还是异步的？
easy 接口是同步。
4.2 是否线程安全？
绝对不应该在线程之间共享同一个libcurl handle，不管是easy handle还是multi handle, libcurl是线程安全的, 其它依赖的库是否线程安全？
4.3 是否可在高并发情况下使用？
4.4 使用何种license(MIT)
4.5 如何设置http 请求的方法(POST, GET or other), 头部，协议版本号(HTTP/1.0 or HTTP/1.1)
4.6 如何获得返回的status code, 头部，协议版本号(HTTP/1.0 or HTTP/1.1)

```
curl_easy_getinfo(curl, CURLINFO_RESPONSE_CODE, &response_code);
curl_easy_setopt(curl, CURLOPT_CUSTOMREQUEST, "SETUP");
curl_easy_setopt(curl, CURLOPT_HTTP_VERSION, CURL_HTTP_VERSION_1_1);
```

4.7
CURLOPT_HEADERFUNCTION，CURLOPT_HEADERDATA
回调函数原型为 size_t function( void *ptr, size_t size,size_t nmemb, void *stream); libcurl一旦接收到http 头部数据后将调用该函数。CURLOPT_WRITEDATA 传递指针给libcurl，该指针表明CURLOPT_HEADERFUNCTION 函数的stream指针的来源


// 注册回调函数
curl_easy_setopt(easy_handle, CURLOPT_READFUNCTION, read_function); 
// 设置自定义指针
curl_easy_setopt(easy_handle, CURLOPT_READDATA, &filedata); 
curl_easy_setopt(easy_handle, CURLOPT_UPLOAD, 1L); // 设置上传文件大小 
curl_easy_setopt(easy_handle, CURLOPT_INFILESIZE_LARGE, file_size);



5. libcurl 几个重要函数
```
curl_easy_setopt(curl, CURLOPT_VERBOSE, 1L);
curl_easy_setopt(curl, CURLOPT_CONNECTTIMEOUT, 2000);
curl_easy_getinfo(curl, CURLINFO_RESPONSE_CODE, &response_code);
curl_slist* headers = NULL;
headers = curl_slist_append(headers, "Content-Type: text/xml");
curl_easy_setopt(curl, CURLOPT_HTTPHEADER, headers);
curl_httppost* post = NULL;
curl_httppost* last = NULL;
curl_formadd(&post, &last, CURLFORM_COPYNAME, "name", CURLFORM_COPYCONTENTS, "zheng", CURLFORM_END);
curl_formadd(&post, &last, CURLFORM_COPYNAME, "project", CURLFORM_COPYCONTENTS, "curl", CURLFORM_END);
curl_easy_setopt(curl, CURLOPT_WRITEFUNCTION, &process_data);
curl_easy_setopt(curl, CURLOPT_WRITEDATA, NULL);

static size_t process_data(void* ptr, size_t size, size_t nmemb, void* param) {
  return size * nmemb;
}

curl_easy_setopt(easy_handle, CURLOPT_USERPWD, "user_name:password");
```

通过将CURLOPT_HTTPGET设为1可以使easy handle回到最原始的状态