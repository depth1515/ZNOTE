# Mockjs小而美的Mock数据神器

## Mockjs

先放上官方地址和文档地址

[mockjs]("http://mockjs.com/")

[mockjs文档]("https://github.com/nuysoft/Mock/wiki")

正如官网介绍，它是个数据类型丰富，用法简单，方便扩展的工具。

能够让我们前端开发人员很方便使用它mock数据。

简单的例子：

```html
<script src="http://mockjs.com/dist/mock.js"></script>
<script>
    
	var data = Mock.mock({
         // 属性 list 的值是一个数组，其中含有 1 到 10 个元素
         'list|1-10': [{
             // 属性 id 是一个自增数，起始值为 1，每次增 1
             'id|+1': 1
         }]
    })
    
    console.log(JSON.stringify(data, null, 2))
    // 这里输出 1 - 10 随机数量数组数据
    // eg：
    /*
    {
   		list: [
            {
                id:1	
            },
            {
                id:2	
            },
            {
                id:3	
            },
    	]
    }
    */
</script>
```

使用 `random`

```html
<script src="http://mockjs.com/dist/mock.js"></script>
<script>
    const random = Mock.Random
    const friends = ["zgz","zgz1","zgz2"]
    const data2 = {
      email: random.email(),
      name: random.cname(),
      // 宽高，背景颜色，字体颜色，图片格式，文字
      iimage: random.image('200x100', '#894FC4', '#FFF', 'debp', 'Mock.js'),
      // 和pick意思一样，在friends里选择一项
      firend: random.pick(friends),
      city: random.city(),
      url: random.url()
    }

    // 输出结果
    console.log(JSON.stringify(data2, null, 2))
</script>
```

用mock模拟get请求，用axios发送请求

```html
<buttom type="button" id="app">button</buttom>
<script src="http://mockjs.com/dist/mock.js"></script>
  <script src="https://cdn.bootcdn.net/ajax/libs/axios/0.21.1/axios.min.js"></script>
  <script>
    var obj = {
      aa:'11',
      bb:'22',
      cc:'33',
      dd:'44'
    }
    Mock.mock('http://api.yourdomain.com/getlist/',"get",{
      'user|3-10':[
        {
          "uid|+1":1,
          name: '@cname',
          "age|18-28":0,
          birthday: '@date("yyyy-MM-dd")',
          city: '@city',
          "fromObj|2": obj
        }
      ]
    })
    document.querySelector('#app').onclick = function(){
      axios.get('http://api.yourdomain.com/getlist/').then(res=>{
        // 查看输出
        console.log(res)
      }).catch(e=>{
        console.log(e)
      })
    }
  </script>
```

### vue项目使用：

```js
// /mock/index.js文件(mock的总引用文件)
const Mock = require('mockjs')
const mocks = require('./mocks')
/* mocks文件：
	// ....
    module.exports = [
        {
            url:'xxx/'
            type: 'get/post...',
            response:()=>{
                return {
                  code: 200,
                  data: {
                    // 这里的list可以用第一个例子的数据或者自己再定义
                    total: list.length,
                    list: list
                  },
                  status: true
                }
            }
        },
        {
			...do something get / post
        },
        ...
    ]
*/
const mocks = [
   ...mocks
]

function mockXHR() {
 for (const i of mocks) {
   // 和上面模拟get请求类似的用法，可以参考官方文档
   Mock.mock(new RegExp(i.url), i.type || 'get', i.response)
 }
}

module.exports = {
 mocks,
 mockXHR
}
```

```js
// main.js (引入并初始化mock接口)
const { mockXHR } = require('./mock')
mockXHR()
```

```js
// getCourseList 发送 ajax 请求
import { getCourseList } from '@/api/course'
getCourseList().then(res => {
    console.log(res.list)
});
```

## 结束

有什么错误或者不足希望大家指出，谢谢观看。

