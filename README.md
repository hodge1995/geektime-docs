## 极客时间文档

极客时间 markdown & pdf 文档

----

* 看 markdown文档，推荐: https://github.com/uaxe/geektime-docs 🌟🌟🌟
* 看 pdf文档，推荐: https://github.com/uaxe/geektime-pdfs 🌟🌟🌟
* 看 音视频，推荐: https://github.com/zkep/my-geektime 🌟🌟🌟🌟🌟

###  markdown 在线文档

 * [netlify](https://geektime-docs.netlify.app/)
 * [github](https://uaxe.github.io/geektime-docs/)


> tips: 在线文档支持 PC 浏览器，也支持移动端浏览器

### 本地部署

#### docker方式
```shell
docker run -d -p 8091:8091 --restart always  --name geektime-docs  zkep/geektime-docs
```
浏览器访问：<http://127.0.0.1:8091/>

#### 源码方式
```shell
git clone https://github.com/uaxe/geektime-docs.git  --depth 1

pip install mkdocs-material

cd geektime-docs/后端-架构/说透中台/

mkdocs serve
```

浏览器访问：<http://127.0.0.1:8000/>


#### 本项目markdown文档全部由 [my-geektime](https://github.com/zkep/my-geektime) 生成


### 问题汇总

#### 1. http Referer导致的裂图，图片不显示 

方案1： 直接看pdf吧 [geektime-pdfs](https://github.com/uaxe/geektime-pdfs)

方案2： VIP用户，部署[my-geektime](https://github.com/zkep/my-geektime)服务，缓存对应的VIP课程 

方案3： 推荐本地使用中间代理人服务，拦截请求，改写 http 请求的 Referer 的思路

[go-mitmproxy](https://github.com/lqqyt2423/go-mitmproxy/blob/main/examples/http-add-header/main.go)

```go 
package main

import (
	"github.com/lqqyt2423/go-mitmproxy/proxy"
	"log"
	"path/filepath"
	"strings"
)

type AddHeader struct {
	proxy.BaseAddon
}

func (a *AddHeader) Requestheaders(f *proxy.Flow) {
	log.Println("AddHeader", f.Request.URL.String())
	host := f.Request.URL.Host
	if strings.Contains(host, ":") {
		host = host[:strings.Index(host, ":")]
	}
	matched, _ := filepath.Match("static001.geekbang.org", host)
	if matched {
		f.Request.Header.Add("Referer", f.Request.URL.String())
	}
}

func main() {
	opts := &proxy.Options{
		Addr:              ":9080",
		StreamLargeBodies: 1024 * 1024 * 5,
	}

	p, err := proxy.NewProxy(opts)
	if err != nil {
		log.Fatal(err)
	}

	p.AddAddon(&AddHeader{})

	log.Fatal(p.Start())
}

```
   
[mitmproxy](https://github.com/mitmproxy/mitmproxy/blob/main/examples/addons/http-add-header.py)
```python
"""Add an HTTP header to each request."""

class AddHeader:
    def __init__(self):
        self.num = 0

    def request(self, flow):
        if flow.request.host.startswith('static001.geekbang.org'):
            flow.request.headers["Referer"] = flow.request.url


addons = [AddHeader()]
```
   

   



