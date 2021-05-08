## Goby API

  goby linux 1.8.230版本API整理及分布式扫描样例

  以下整理自goby linux 1.8.230版本，可利用goby api实现自定义指纹/漏洞扫描和分布式扫描等，更多api持续补充

  下列api基于账号密码为空，即每次请求api中存在请求头`Authorization: Basic Og==`

  ### 下发指纹识别任务

- 即新建扫描任务->漏洞->禁用。若扫描对象为域名，则将请求body中ips字段值替换为域名列表

- 请求：

  方法：POST

  url：http://x.x.x.x:8361/api/v1/startScan

  Header:

  ```
  POST /api/v1/startScan
  Content-Type: application/json;charset=UTF-8
  Content-Length: 613
  Authorization: Basic Og==
  Host: x.x.x.x:8361
  Connection: close
  ```

  body：

  1. taskName: 任务名
  2. asset->ips: 扫描对象列表，遵循goby扫描对象格式，单个goby api扫描对象数量经实测，建议小于1000，否则容易导致oom或goby扫描任务处于假死状态

  2. asset->ports: 扫描目标端口，**仅能为字符串格式**
  3. options->rate: 扫描速度

  ```
  {
  	"taskName": "api test",
  	"asset": {
  		"ips": ["1.1.1.1"],
  		"ports": "1-65535"
  	},
  	"vulnerability": {
  		"type": "-1",
  		"pocs_hosts": null
  	},
  	"options": {
  		"queue": 0,
  		"random": true,
  		"rate": 100,
  		"portscanmode": 0,
  		"disableMdns": false,
  		"disableUpnp": false,
  		"CheckHoneyPot": false,
  		"enableCrawler": false,
  		"crawlerScope": 0,
  		"crawlerConcurrent": 5,
  		"crawlerMaxLinks": 50,
  		"crawlerMaxCrawlLinks": 1000,
  		"connectionSize": 200,
  		"screenshotRDP": false,
  		"screenshot": false,
  		"deepAnalysis": true,
  		"extracthost": false,
  		"fofaFetchSubdomainEnabled": false,
  		"fofaEmail": "",
  		"fofaKey": "",
  		"fofaFetchSize": 100,
  		"pingFirst": false,
  		"pingCheckSize": 10,
  		"pingConcurrent": 2,
  		"pingSendCount": 2
  	}
  }
  
  ```

  - 返回：

    若请求中有参数值格式错误，同样会返回以下数据，但扫描进度会秒变为100%

     **获得的taskId字段值将作为后续所有任务相关api请求参数**

  ```
  HTTP/1.1 200 OK
  Content-Type: application/json
  Date: Sun, 28 Mar 2021 10:43:17 GMT
  Content-Length: 67
  Connection: close
  
  {"statusCode":200,"messages":"","data":{"taskId":"20210328184317"}}
  ```

  ### 获取扫描进度

- 请求：

  方法：POST

  url：http://x.x.x.x:8361/api/v1/getProgress

  header:

  ```
  POST /api/v1/getProgress HTTP/1.1
  Content-Type: application/json;charset=UTF-8
  Content-Length: 27
  Authorization: Basic Og==
  Host: x.x.x.x:8361
  Connection: close
  ```

  body：

  ```
  {"taskId":"20210504113307"}
  ```

- 未完成任务返回

  1. state字段：

     1: 正在扫描

     2: 已完成

     4: 已暂停

  2. progress字段: 代表扫描进度百分比：如9即9%

  ```
  HTTP/1.1 200 OK
  Content-Type: application/json
  Date: Sun, 28 Mar 2021 10:43:17 GMT
  Content-Length: 138
  Connection: close
  
  {
      "statusCode": 200,
      "messages": "",
      "data": {
          "logs": [
              {
                  "type": 3,
                  "json": {
                      "all": 1,
                      "host": "1.1.1.1",
                      "position": 1
                  }
              }
          ],
          "progress": 9,
          "state": 1
      }
  }
  ```

  

  - 已完成任务返回

  ```
  HTTP/1.1 200 OK
  Content-Type: application/json
  Date: Sun, 28 Mar 2021 10:43:41 GMT
  Content-Length: 78
  Connection: close
  
  {"statusCode":200,"messages":"","data":{"logs":null,"progress":100,"state":2}}
  ```

  ### 任务结果获取

  - 请求

    方法: POST

    url: http://x.x.x.x:8361/api/v1/getValueCategory

    header:

    ```
    POST /api/v1/assetSearch HTTP/1.1
    Content-Type: application/json;charset=UTF-8
    Content-Length: 128
    Authorization: Basic Og==
    Host: x.x.x.x:8361
    Connection: close
    ```

    body:

   ```
  {"taskId": "20210328184317"}
   ```

  - 返回

  ```
  {
  	"statusCode": 200,
  	"messages": "",
  	"data": {
  		"taskId": "20210328184317",
  		"query_total": {
  			"ips": 1,
  			"ports": 1,
  			"protocols": 1,
  			"assets": 1,
  			"vulnerabilities": 0,
  			"dist_ports": 1,
  			"dist_protocols": 1,
  			"dist_assets": 1,
  			"dist_vulnerabilities": 0
  		},
  		"total": {
  			"assets": 1,
  			"ips": 1,
  			"ports": 1,
  			"vulnerabilities": 0,
  			"allassets": 0,
  			"allips": 0,
  			"allports": 0,
  			"allvulnerabilities": 0,
  			"scan_ips": 0,
  			"scan_ports": 0
  		},
  		"ips": [{
  			"ip": "1.1.1.1",
  			"mac": "",
  			"os": "",
  			"hostname": "",
  			"honeypot": "0",
  			"ports": [{
  				"port": "22",
  				"baseprotocol": "tcp"
  			}],
  			"protocols": {
  				"1.1.1.1:22": {
  					"port": "22",
  					"hostinfo": "1.1.1.1:22",
  					"url": "",
  					"product": "OpenSSH",
  					"protocol": "ssh",
  					"json": "",
  					"products": ["OpenSSH"],
  					"protocols": ["ssh"]
  				}
  			},
  			"tags": [{
  				"rule_id": "7512",
  				"product": "OpenSSH",
  				"company": "xxx",
  				"level": "3",
  				"category": "Other Support System",
  				"parent_category": "Support System",
  				"soft_hard": "2",
  				"version": "7.4"
  			}],
  			"vulnerabilities": null,
  			"screenshots": null,
  			"favicons": null,
  			"hostnames": [""]
  		}],
  		"products": {
  			"software": {
  				"total_assets": 1,
  				"risk_assets": 0,
  				"lists": [{
  					"name": "OpenSSH",
  					"company": "xxx",
  					"total_assets": 1,
  					"risk_assets": 0
  				}]
  			},
  			"hardware": {
  				"total_assets": 0,
  				"risk_assets": 0,
  				"lists": null
  			}
  		},
  		"companies": {
  			"software": {
  				"total_assets": 1,
  				"risk_assets": 0,
  				"lists": [{
  					"name": "xxx",
  					"total_assets": 1,
  					"risk_assets": 0
  				}]
  			},
  			"hardware": {
  				"total_assets": 0,
  				"risk_assets": 0,
  				"lists": null
  			}
  		}
  	}
  }
  ```

### 任务停止

- 请求

  方法: POST

  url: http://x.x.x.x:8361/api/v1/stopScan

  header:

  ```
  POST /api/v1/stopScan HTTP/1.1
  Content-Type: application/json;charset=UTF-8
  Content-Length: 27
  Authorization: Basic Og==
  Host: x.x.x.x:8361
  Connection: close
  ```

  body:

```
{"taskId":"20210504113307"}
```

### 任务恢复

- 请求

  方法: POST

  url: http://x.x.x.x:8361/api/v1/resumeScan

  header:

  ```
  POST /api/v1/resumeScan HTTP/1.1
  Content-Type: application/json;charset=UTF-8
  Content-Length: 150
  Authorization: Basic Og==
  Host: x.x.x.x:8361
  Connection: close
  ```

  body:

```
{"taskId":"20210504113307"}
```

- 返回

  ```
  HTTP/1.1 200 OK
  Content-Type: application/json
  Date: Tue, 04 May 2021 03:36:04 GMT
  Content-Length: 44
  Connection: close
  
  {"statusCode":200,"messages":"","data":null}
  ```

### 任务删除

​	**若要删除任务，则任务必须处于已完成或已停止状态。**

​ **任务删除接口响应比其他接口略长，建议请求timeout至少给到3秒以上或走异步请求**

- 请求

  方法: POST

  url: http://x.x.x.x:8361/api/v1/deleteTask

  header:

  ```
  POST /api/v1/deleteTask HTTP/1.1
  Content-Type: application/json;charset=UTF-8
  Content-Length: 27
  Authorization: Basic Og==
  Host: x.x.x.x:8361
  Connection: close
  ```

  body:

```
{"taskId":"20210328184317"}
```

- 返回

```
HTTP/1.1 200 OK
Content-Type: application/json
Date: Sun, 28 Mar 2021 11:50:33 GMT
Content-Length: 44
Connection: close

{"statusCode":200,"messages":"","data":null}
```

### 任务列表

  由于单个goby api只能同时进行一个扫描任务，可通过该接口判断当前任务列表中是否有正在扫描的任务（即该goby api是否空闲）及是否假死，具体字段为state和progress。

- 请求

  方法: GET

  url: http://x.x.x.x:8361/api/v1/tasks 

  header:

  ```
  GET /api/v1/tasks HTTP/1.1
  Content-Type: application/json;charset=UTF-8
  Authorization: Basic Og==
  Host: x.x.x.x:8361
  Connection: close
  ```

- 返回

```
HTTP/1.1 200 OK
Content-Type: application/json
Date: Sun, 28 Mar 2021 11:57:36 GMT
Connection: close
Transfer-Encoding: chunked

{"statusCode":200,"messages":"","data":[{"taskId":"20210328195307","name":"","created_time":"2021-03-28 19:53:07","end_time":"","targets":"1.1.1.1","ports":"1-65535","state":1,"progress":19,"memo":"{\"taskName\":\"api test\",\"taskId\":null,\"asset\":{\"ips\":[\"1.1.1.1\"],\"ports\":\"1-65535\"},\"vulnerability\":{\"type\":\"-1\",\"pocs_hosts\":null},\"options\":{\"queue\":0,\"rate\":100,\"random\":true,\"interface\":\"\",\"portScanMode\":0,\"proxy\":\"\",\"connectionSize\":200,\"screenshot\":false,\"screenshotRDP\":false,\"extractHost\":false,\"disableMdns\":false,\"disableUpnp\":false,\"fofaFetchSubdomainEnabled\":false,\"bruteforceSubdomainEnabled\":false,\"fofaKey\":\"\",\"fofaEmail\":\"\",\"fofaFetchSize\":100,\"pingFirst\":false,\"pingCheckSize\":10,\"pingConcurrent\":2,\"pingSendCount\":2,\"deepAnalysis\":true,\"scanICMP\":false,\"scanTreck\":false,\"checkHoneyPot\":false,\"enableCrawler\":false,\"crawlerScope\":0,\"crawlerConcurrent\":5,\"crawlerMaxLinks\":50,\"crawlerMaxCrawlLinks\":1000}}","total":{"assets":0,"ips":0,"ports":0,"vulnerabilities":0,"allassets":0,"allips":0,"allports":0,"allvulnerabilities":0,"scan_ips":0,"scan_ports":65535},"agenttaskid":""},{"taskId":"20210328194216","name":"","created_time":"2021-03-28 19:42:16","end_time":"2021-03-28 19:42:39","targets":"1.2.9.45","ports":"22","state":2,"progress":100,"memo":"{\"taskName\":\"api test\",\"taskId\":null,\"asset\":{\"ips\":[\"1.2.9.45\"],\"ports\":\"22\"},\"vulnerability\":{\"type\":\"-1\",\"pocs_hosts\":null},\"options\":{\"queue\":0,\"rate\":100,\"random\":true,\"interface\":\"\",\"portScanMode\":0,\"proxy\":\"\",\"connectionSize\":200,\"screenshot\":false,\"screenshotRDP\":false,\"extractHost\":false,\"disableMdns\":false,\"disableUpnp\":false,\"fofaFetchSubdomainEnabled\":false,\"bruteforceSubdomainEnabled\":false,\"fofaKey\":\"\",\"fofaEmail\":\"\",\"fofaFetchSize\":100,\"pingFirst\":false,\"pingCheckSize\":10,\"pingConcurrent\":2,\"pingSendCount\":2,\"deepAnalysis\":true,\"scanICMP\":false,\"scanTreck\":false,\"checkHoneyPot\":false,\"enableCrawler\":false,\"crawlerScope\":0,\"crawlerConcurrent\":5,\"crawlerMaxLinks\":50,\"crawlerMaxCrawlLinks\":1000}}","total":{"assets":1,"ips":1,"ports":1,"vulnerabilities":0,"allassets":1,"allips":1,"allports":1,"allvulnerabilities":0,"scan_ips":0,"scan_ports":1},"agenttaskid":""}]}
```


### 分布式扫描

多个goby api agent配合合理的任务调度策略，即可实现分布式指纹识别/漏扫:
![image](https://user-images.githubusercontent.com/26686336/116987827-1e033a80-ad02-11eb-8340-a1a793248122.png)
![image](https://user-images.githubusercontent.com/26686336/116987964-5145c980-ad02-11eb-81ca-10c51cb1a494.png)

