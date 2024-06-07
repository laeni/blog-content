老系统中出现代码，如下所示：

```java
Client client = JaxWsDynamicClientFactory.newInstance().createClient("http://xxx:8081/xxx/service/merchantCloseMgr?wsdl");
Object[] res = client.invoke("updateMerchantInfo", xmlReq);
```

这东西升级后Java后这里不对那里不对的，看上面的地址应该是基于HTTP协议的，所以最终打算更改为HTTP直接调用。

1. 访问HTTP接口`http://xxx:8081/xxx/service/merchantCloseMgr?wsdl`得到XML返回值，并将返回值写入文件（如`xxx.wsdl.xml`）。

2. 安装并打开桌面软件**SoapUI**（类似*Postman*的工具）。

3. 点击**SOAP**（创建SOAP项目），`Project Name`随意，`Initial WSDL`处选择刚刚的XML文件（如果可以直接访问上面的http地址的话，这里直接填写地址也行），填写后点击`OK`就会自动解析出该项目下所有接口。

4. 展开需要使用的接口，双击`Request 1`即可出现请求地址以及请求体。

   ![image-20230920180419274](./wsdl2http.assets/image-20230920180419274.png)

5. 将需要请求内容替换示例中的**问号**后以POST方式请求即可。

   注意：一般请求的内容为XML，所有不能直接将请求内容放在那里，而是需要进行HTML编码或者放在`<![CDATA[]]>`中;

