# koo-node-dubbo
nodejs通过dubbo默认协议通信,参照node-zookeeper-dubbo版本2.2.2

1.解决Not found exported service的问题

    dubbo提供的provides里的有2个bean,dubbo会自动把第2个bean加上2，导致service name的interface和provider url的interface不一致

2.node-zookeeper-dubbo的3.x版本有好多bug，已知bug：

    no available connection
    (node:11892) Warning: Possible EventEmitter memory leak detected. 11 notification listeners added. Use emitter.setMaxListeners() to increase limit 
    ...
    


### Usage

```javascript
const nzd=require('koo-node-dubbo');
const app=require('express')();
const opt={
  application:{name:'fxxk'},
  register:'www.cctv.com:2181',
  dubboVer:'2.5.3.6',
  root:'dubbo',
  dependencies:{
    Foo:{
      interface:'com.service.Foo',
      version:'LATEST',
      timeout:6000,
      group:'isis',
      methodSignature: {
        findById : (id) => [ {'$class': 'java.lang.Long', '$': id} ],
        findByName : (name) => (java) => [ java.String(name) ],
      }
    },
    Bar:{
      interface:'com.service.Bar',
      version:'LATEST',
      timeout:6000,
      group:'gcd'
    }
  }  
}
opt.java = require('js-to-java')

const Dubbo=new nzd(opt);

const customerObj = {
  $class: 'com.xxx.XXXDTO',
  $: {
    a: 1,
    b: 'test',
    c: {$class: 'java.lang.Long', $: 123}
  }
};

app.get('/foo',(req,res)=>{
  Dubbo.Foo
    .xxMethod({'$class': 'java.lang.Long', '$': '10000000'},customerObj)
    .then(data=>res.send(data))
    .catch(err=>res.send(err))
})

app.get('/foo/findById',(req,res)=>{
  Dubbo.Foo
    .findById(10000)
    .then(data=>res.send(data))
    .catch(err=>res.send(err))
})

app.listen(9090)



```
### 注意

须等待初始化完毕才能正常使用，标志：**Dubbo service init done**

如果要和1.x版本共存的话试试这个，[niv](https://github.com/scott113341/npm-install-version).

### 参数配置说明
- **application**
  * name - 项目名称，必填
- **register** - zookeeper服务地址，必填
- **dubboVer** - dubbo版本，必填
- **root** - 注册到zookeeper上的根节点名称
- **dependencies** - 依赖的服务集，必填
  * Foo - 自定义名称，这里方便起见用Foo作为事例，必填
    * interface - 服务地址，必填
    * version - 注册的服务版本
    * timeout	-	超时时间，默认6000
    * group	-	分组
    * methodSignature	-	方法签名

可以选择使用  [js-to-java](https://github.com/node-modules/js-to-java)， 能极大提高效率
```javascript
var arg={$class:'int',$:123};
//等同于
var arg=java('int',123);
```


[npm-image]:http://img.shields.io/npm/v/node-zookeeper-dubbo.svg?style=flat-square
[npm-url]:https://npmjs.org/package/node-zookeeper-dubbo?style=flat-square
[downloads-image]:http://img.shields.io/npm/dm/node-zookeeper-dubbo.svg?style=flat-square
