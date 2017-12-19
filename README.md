这是一个简单的node-server,实现了简单的路由和静态页面处理。

## 开启服务器
打开命令终端，切换到node-server目录下，输入`node server.js`启动我们的服务器。我们监听的是本机8080端口，在浏览器中输入`http://127.0.0.1:8080`或者`http://localhost:8080`就可以访问我们的服务器的`/`。

## 路由处理
服务器引入了一个路由处理函数`routePath`，可以通过浏览器中发送的请求的URL解析出对应的网站路由，并调用函数处理响应。对象`routes`中定义了网站的其他页面路径和对应的处理函数，可以自行配置。
```
var server = http.createServer(function(req, res){
  routePath(req, res)
})
// 路由配置对象
var routes = {
  '/a': function(req, res){
    res.end(JSON.stringify(req.query))
  },
  '/b': function(req, res){
    res.end('match /b')
  },
  '/a/c': function(req, res){
    res.end('match /a/c')
  },
  '/search': function(req, res){
    res.end('username=' + req.body.username + ',password=' + req.body.password)
  }
}
// 路由对象调用
function routePath(req, res){
  var pathObj = url.parse(req.url,true)

  var handleFn = routes[pathObj.pathname]
  if(handleFn){
    req.query =  pathObj.query

    var body = '';
    req.on('data', function(chunk){
      body += chunk
    }).on('end', function(){
      req.body = parseBody(body)
      handleFn(req.res)
    })
  }else{
    staticRoot(path.resolve(__dirname, 'sample'), req, res)
  }
}
// post请求体处理，将请求体转换为`obj`对象
function parseBody(body){
  console.log(body)
  var obj = {}
  body.split('&').forEach(function(str){
    obj[str.split('=')[0]] = str.split('=')[1]
  })
  return obj
}
```
## 静态页面
当输入的是静态页面时，服务器会调用`staticRoot`函数进行处理。这里我们在浏览器中输入
`http://localhost:8080/test.html`可以访问我们的例子。

```

function staticRoot(staticPath, req, res){
  var pathObj = url.parse(req.url, true)
  var filePath = path.join(staticPath, pathObj.pathname)
  fs.readFile(filePath, 'binary', function(err, content){
    if(err){
      res.writeHead(404, 'Not Found')
      return res.end()
    }
    res.writeHead(200, 'ok')
    res.write(content, 'binary')
    res.end()
  })
}
```