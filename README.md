
# aliddns4dsm

## 介绍

如果使用了阿里云（万网）的域名解析服务的话，那么就可以通过它提供的API，使用HTTP访问动态修改解析地址，以实现DDNS的功能。阿里云也提供了一些语言的SDK，但是并没有Shell版本的。

此`shell`是在**jwping**的**AliDDNS**基础上针对群晖而改了自用的，因为群晖默认不支持 `band-dig`,需要一些修改。如需应用更多场景，请前往 [jwping/AliDDNS](https://github.com/jwping/AliDDNS) 了解更多。

注意：此脚本只实现了调用**修改域名解析记录**和**获取解析记录列表**的API的功能，并没有完整实现整个SDK。但是因为脚本已经实现了API的签名机制，所以很容易实现其他API的调用。

参考：[阿里云解析API文档](https://help.aliyun.com/document_detail/29739.html)


## 功能

* 仅在当前IP地址和域名解析设置不同时，发起更新请求。（本机当前IP地址通过[3322.org提供的API](http://members.3322.org/dyndns/getip) 进行查询，域名的解析设置通过*nslookup*进行查询。）
* 查看账户下所有的域名信息。

## 使用方法

### 1. 修改基础配置
修改脚本的`ipv4`或`ipv6`函数，其中`DomainRecordId`不清楚的话暂时不用修改，如:
```sh
#Accesskey在开头Setting代码段中修改。
AccessKeyId="MyID"
AccessKeySec="MySecret"
```

此处的DNS解析服务器为万网，如无特殊需求可以不做修改。
```
DNSServer="dns9.hichina.com"
```

修改*Domain*在尾部ipv4或ipv6的函数调用中修改，*DomainRR*为需要监控的二级域名，*DomainName*为需要监控的域名，*NslookupDrefix* 用于截取域名记录的IP地址时ipv4和ipv6的区分。
```sh
DomainRecordId="00000"
DomainRR="www"
DomainName="example.com"
DomainType="A"
NslookupDrefix="Address: "
```

### 2. 查看DomainRecordId
如果不清楚DomainRecordId的话，修改`ipv4`或`ipv6`函数，在里面调用`describe_record`，如：
```sh
	ipv6()
	{
		describe_record
		#update_record
	}
```
然后执行这个脚本。如果没问题的话，就能获取到域名的所有解析记录的列表了：
```JSON
{"PageNumber":1,"TotalCount":1,"PageSize":1,"RequestId":"0000","DomainRecords":
  {"Record":[{"RR":"www","Status":"ENABLE","Value":"8.8.8.8",
  "RecordId":"21332133","Type":"A","DomainName":"example.com",
  "Locked":false,"Line":"default","TTL":"600"},]}
  }HttpCode:200
```
上面的结果中，RecordId为*21332133*。得到结果后再修改`DomainRecordId`为正确的值。
  
### 3. 执行脚本
基础配置完成后，改回原入口函数：
```sh
	ipv6()
	{
		#describe_record
		update_record
	}
```
执行脚本即可，脚本会在本机IP地址和当前域名解析设置不同的时候调用API更新设置。
### 4. 选择执行函数
如下选择需要执行的函数，我因为没有IPV4，所以只启用了IPV6
```sh
ipv6
#ipv4
```
### 5. 在群晖中设置计划任务
网上很多，请自行查找下，略。