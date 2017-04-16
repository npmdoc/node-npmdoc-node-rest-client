# api documentation for  [node-rest-client (v3.1.0)](https://github.com/aacerox/node-rest-client)  [![npm package](https://img.shields.io/npm/v/npmdoc-node-rest-client.svg?style=flat-square)](https://www.npmjs.org/package/npmdoc-node-rest-client) [![travis-ci.org build-status](https://api.travis-ci.org/npmdoc/node-npmdoc-node-rest-client.svg)](https://travis-ci.org/npmdoc/node-npmdoc-node-rest-client)
#### node API REST client

[![NPM](https://nodei.co/npm/node-rest-client.png?downloads=true&downloadRank=true&stars=true)](https://www.npmjs.com/package/node-rest-client)

[![apidoc](https://npmdoc.github.io/node-npmdoc-node-rest-client/build/screenCapture.buildCi.browser.%252Ftmp%252Fbuild%252Fapidoc.html.png)](https://npmdoc.github.io/node-npmdoc-node-rest-client/build/apidoc.html)

![npmPackageListing](https://npmdoc.github.io/node-npmdoc-node-rest-client/build/screenCapture.npmPackageListing.svg)

![npmPackageDependencyTree](https://npmdoc.github.io/node-npmdoc-node-rest-client/build/screenCapture.npmPackageDependencyTree.svg)



# package.json

```json

{
    "author": {
        "name": "Alejandro Alvarez Acero"
    },
    "bugs": {
        "url": "https://github.com/aacerox/node-rest-client/issues"
    },
    "dependencies": {
        "debug": "~2.2.0",
        "follow-redirects": ">=1.2.0",
        "xml2js": ">=0.2.4"
    },
    "description": "node API REST client",
    "devDependencies": {
        "mocha": "*",
        "should": ">= 0.0.1"
    },
    "directories": {},
    "dist": {
        "shasum": "e0beb6dda7b20cc0b67a7847cf12c5fc419c37c3",
        "tarball": "https://registry.npmjs.org/node-rest-client/-/node-rest-client-3.1.0.tgz"
    },
    "engines": {
        "node": "*"
    },
    "gitHead": "082c3ff4f99e4b9285e26bb81f6d344c012cbbd8",
    "homepage": "https://github.com/aacerox/node-rest-client",
    "license": "MIT",
    "main": "./lib/node-rest-client",
    "maintainers": [
        {
            "name": "acero"
        }
    ],
    "name": "node-rest-client",
    "optionalDependencies": {},
    "repository": {
        "type": "git",
        "url": "git+https://github.com/aacerox/node-rest-client.git"
    },
    "scripts": {
        "test": "mocha"
    },
    "version": "3.1.0"
}
```



# <a name="apidoc.tableOfContents"></a>[table of contents](#apidoc.tableOfContents)

#### [module node-rest-client](#apidoc.module.node-rest-client)
1.  [function <span class="apidocSignatureSpan">node-rest-client.</span>Client (options)](#apidoc.element.node-rest-client.Client)

#### [module node-rest-client.Client](#apidoc.module.node-rest-client.Client)
1.  [function <span class="apidocSignatureSpan">node-rest-client.</span>Client (options)](#apidoc.element.node-rest-client.Client.Client)
1.  [function <span class="apidocSignatureSpan">node-rest-client.Client.</span>super_ ()](#apidoc.element.node-rest-client.Client.super_)



# <a name="apidoc.module.node-rest-client"></a>[module node-rest-client](#apidoc.module.node-rest-client)

#### <a name="apidoc.element.node-rest-client.Client"></a>[function <span class="apidocSignatureSpan">node-rest-client.</span>Client (options)](#apidoc.element.node-rest-client.Client)
- description and source-code
```javascript
Client = function (options){
    var self = this,
     // parser response manager
     parserManager = require("./nrc-parser-manager")(),
     serializerManager = require("./nrc-serializer-manager")(),
    // connection manager
    connectManager = new ConnectManager(this, parserManager),
    // io facade to parsers and serailiazers
    ioFacade = function(parserManager, serializerManager){
        // error execution context
        var  errorContext = function(logic){
            return function(){
                try{
                    return logic.apply(this, arguments);
                }catch(err){
                    self.emit('error',err);
                }
              };
        },
        result={"parsers":{}, "serializers":{}};

    // parsers facade
    result.parsers.add = errorContext(parserManager.add);
    result.parsers.remove = errorContext(parserManager.remove);
    result.parsers.find = errorContext(parserManager.find);
    result.parsers.getAll = errorContext(parserManager.getAll);
    result.parsers.getDefault = errorContext(parserManager.getDefault);
    result.parsers.clean = errorContext(parserManager.clean);

    // serializers facade
    result.serializers.add = errorContext(serializerManager.add);
    result.serializers.remove = errorContext(serializerManager.remove);
    result.serializers.find = errorContext(serializerManager.find);
    result.serializers.getAll = errorContext(serializerManager.getAll);
    result.serializers.getDefault = errorContext(serializerManager.getDefault);
    result.serializers.clean = errorContext(serializerManager.clean);

    return result;

    }(parserManager,serializerManager),
    // declare util constants
    CONSTANTS={
        HEADER_CONTENT_LENGTH:"Content-Length"
    };


    self.options = options || {},
    self.useProxy = (self.options.proxy || false)?true:false,
    self.useProxyTunnel = (!self.useProxy || self.options.proxy.tunnel===undefined)?false:self.options.proxy.tunnel,
    self.proxy = self.options.proxy,
    self.connection = self.options.connection || {},
    self.mimetypes = self.options.mimetypes || {},
    self.requestConfig = self.options.requestConfig || {},
    self.responseConfig = self.options.responseConfig || {};

    // namespaces for methods, parsers y serializers
    this.methods={};
    this.parsers={};
    this.serializers={};

    // Client Request to be passed to ConnectManager and returned
    // for each REST method invocation
    var ClientRequest =function(){
        events.EventEmitter.call(this);
    };


    util.inherits(ClientRequest, events.EventEmitter);


    ClientRequest.prototype.end = function(){
        if(this._httpRequest) {
            this._httpRequest.end();
        }
    };

    ClientRequest.prototype.setHttpRequest=function(req){
        this._httpRequest = req;
    };



    var Util = {
       createProxyPath:function(url){
        var result = url.host;
         // check url protocol to set path in request options
         if (url.protocol === "https:"){
            // port is set, leave it, otherwise use default https 443
            result = (url.host.indexOf(":") == -1?url.hostname + ":443":url.host);
        }

        return result;
    },
    createProxyHeaders:function(url){
        var result ={};
        // if proxy requires authentication, create Proxy-Authorization headers
        if (self.proxy.user && self.proxy.password){
            result["Proxy-Authorization"] = "Basic " + new Buffer([self.proxy.user,self.proxy.password].join(":")).toString("base64
");
        }
        // no tunnel proxy connection, we add the host to the headers
        if(!self.useProxyTunnel)
            result["host"] = url.host;

        return result;
    },
    createConnectOptions:function(connectURL, connectMethod){
        debug("connect URL = ", connectURL);
        var url = urlParser.parse(connectURL),
        path,
        result={},
        protocol = url.protocol.indexOf(":") == -1?u ...
```
- example usage
```shell
n/a
```



# <a name="apidoc.module.node-rest-client.Client"></a>[module node-rest-client.Client](#apidoc.module.node-rest-client.Client)

#### <a name="apidoc.element.node-rest-client.Client.Client"></a>[function <span class="apidocSignatureSpan">node-rest-client.</span>Client (options)](#apidoc.element.node-rest-client.Client.Client)
- description and source-code
```javascript
Client = function (options){
    var self = this,
     // parser response manager
     parserManager = require("./nrc-parser-manager")(),
     serializerManager = require("./nrc-serializer-manager")(),
    // connection manager
    connectManager = new ConnectManager(this, parserManager),
    // io facade to parsers and serailiazers
    ioFacade = function(parserManager, serializerManager){
        // error execution context
        var  errorContext = function(logic){
            return function(){
                try{
                    return logic.apply(this, arguments);
                }catch(err){
                    self.emit('error',err);
                }
              };
        },
        result={"parsers":{}, "serializers":{}};

    // parsers facade
    result.parsers.add = errorContext(parserManager.add);
    result.parsers.remove = errorContext(parserManager.remove);
    result.parsers.find = errorContext(parserManager.find);
    result.parsers.getAll = errorContext(parserManager.getAll);
    result.parsers.getDefault = errorContext(parserManager.getDefault);
    result.parsers.clean = errorContext(parserManager.clean);

    // serializers facade
    result.serializers.add = errorContext(serializerManager.add);
    result.serializers.remove = errorContext(serializerManager.remove);
    result.serializers.find = errorContext(serializerManager.find);
    result.serializers.getAll = errorContext(serializerManager.getAll);
    result.serializers.getDefault = errorContext(serializerManager.getDefault);
    result.serializers.clean = errorContext(serializerManager.clean);

    return result;

    }(parserManager,serializerManager),
    // declare util constants
    CONSTANTS={
        HEADER_CONTENT_LENGTH:"Content-Length"
    };


    self.options = options || {},
    self.useProxy = (self.options.proxy || false)?true:false,
    self.useProxyTunnel = (!self.useProxy || self.options.proxy.tunnel===undefined)?false:self.options.proxy.tunnel,
    self.proxy = self.options.proxy,
    self.connection = self.options.connection || {},
    self.mimetypes = self.options.mimetypes || {},
    self.requestConfig = self.options.requestConfig || {},
    self.responseConfig = self.options.responseConfig || {};

    // namespaces for methods, parsers y serializers
    this.methods={};
    this.parsers={};
    this.serializers={};

    // Client Request to be passed to ConnectManager and returned
    // for each REST method invocation
    var ClientRequest =function(){
        events.EventEmitter.call(this);
    };


    util.inherits(ClientRequest, events.EventEmitter);


    ClientRequest.prototype.end = function(){
        if(this._httpRequest) {
            this._httpRequest.end();
        }
    };

    ClientRequest.prototype.setHttpRequest=function(req){
        this._httpRequest = req;
    };



    var Util = {
       createProxyPath:function(url){
        var result = url.host;
         // check url protocol to set path in request options
         if (url.protocol === "https:"){
            // port is set, leave it, otherwise use default https 443
            result = (url.host.indexOf(":") == -1?url.hostname + ":443":url.host);
        }

        return result;
    },
    createProxyHeaders:function(url){
        var result ={};
        // if proxy requires authentication, create Proxy-Authorization headers
        if (self.proxy.user && self.proxy.password){
            result["Proxy-Authorization"] = "Basic " + new Buffer([self.proxy.user,self.proxy.password].join(":")).toString("base64
");
        }
        // no tunnel proxy connection, we add the host to the headers
        if(!self.useProxyTunnel)
            result["host"] = url.host;

        return result;
    },
    createConnectOptions:function(connectURL, connectMethod){
        debug("connect URL = ", connectURL);
        var url = urlParser.parse(connectURL),
        path,
        result={},
        protocol = url.protocol.indexOf(":") == -1?u ...
```
- example usage
```shell
n/a
```

#### <a name="apidoc.element.node-rest-client.Client.super_"></a>[function <span class="apidocSignatureSpan">node-rest-client.Client.</span>super_ ()](#apidoc.element.node-rest-client.Client.super_)
- description and source-code
```javascript
function EventEmitter() {
  EventEmitter.init.call(this);
}
```
- example usage
```shell
n/a
```



# misc
- this document was created with [utility2](https://github.com/kaizhu256/node-utility2)
