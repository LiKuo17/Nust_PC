## 打包规范

1、打包上线

````js
前端：
1、检查env.js 线上链接是否正确
2、package.json的script标签的start 地址是否为线上链接
3、npm run build:prod    打生产环境的包
4、到项目目录，除了node_modules依赖包，其他Ctrl + V 复制，桌面创建一个文件夹，将复制的文件放进去，压缩后发给后端
后端：
5、后端收到后将所有文件放到nginx下的官网项目下，后端在这个项目下需要装依赖
   1、npm i    装node_modules依赖
      注：如果已经有依赖包则不需要npm i 安装了
   2、用pm2 list 查到当前pm2正在守护的进程列表，如果有，将所有进程删除掉
     备注：为什么要删除呢？因为线上官网部署只能保留一个。
            如何删除进程：
          pm2 list           先查看当前的进程列表
       pm2 delete 数字    删除指定进程，数字对应进程列表的第一个、第N个
              示例：pm2 delete 1
        pm2 list 进程为空后执行下一个步骤
   3、npm run pm2   启动pm2守护部署后的项目进程，pm2命令包含了pm2守护部署进程和启动项目
   4、pm2守护进程的status为online,代表部署完成
````

2、本地启动

```js
npm run dev    // 本地 
记得检查env.js的dev地址是否为本地服务器地址，以及package.json的script的dev BASE_URL是否为本地服务器地址

npm start  // 本地模拟线上版本    BASE_URL记得更改为线上的
```

3、Nginx和pm2常用命令

````js
Nginx 
  nginx -s reload   //  修改配置文件后需要重载   
  nginx -s  start   //  启动
  nginx  -s stop    //  关闭

pm2
npm i pm2 -g   // 安装 
npm run pm2    // 启动
pm2 list       // 查看进程列表
pm2 stop 数字   // 暂停制定进程
````

4、打包上线后前端报404错误

简言之：
前端新装了sitemap插件依赖后，后端在打包时需要 删掉老node_modules重装 或者 直接npm i覆盖 依赖包,让node_modules中的依赖拥有package.json中所装的依赖，这样就能百分百保证前端的代码和Nginx服务器上前端的代码一致。

详细原理解析：
有一次前端新增sitemap站点地图插件后打包到Nginx服务器，但是放到服务器后上线报404错误，根据回溯当天前端开发的代码发现，前端新增了sitemap依赖后同时也将该插件自动添加到package.json中，由于每次前端打包都需要把package.json放到Nginx服务器，Nginx服务器启动前端项目也需要检测package.json的依赖是否存在于node_modules中，是的话Nginx服务器前端项目可以正常运行，node_modules中一旦缺少package.json中的部分依赖后则会报错404(至于Nginx启动时为啥没有像前端那样报缺少依赖的warnning提示这个不清楚)，但是此时只要Nginx服务器的前端项目 删掉老node_modules重装 或者 直接npm i覆盖 依赖包就可以了。

定位问题：
通过从时间轴上反推在出现这个bug之前前后端代码都做了哪些操作(哪怕是微小更改的部分也需要重新检查两遍，根据逻辑慢慢排查，直到定位问题)，从而为定位问题的过程提供一定的加速度。
（题外话：事实上生活中很多事情如果出现问题也可以从时间轴反推得出根源问题，例如上火了可以推测在往日的日子里饮食作息是否规范）

## 如何生成sitemap.xml文件

  如果每天官网的商品有变动(CRUD)，那么需要在下班前重新生成一份新的sitemap.xml站点地图发给后端更新到服务器上，同时也得通知负责SEO的同学把  <https://www.iliving.cn/sitemap.xml>  这个链接在百度资源后台收录提交一下，让网站的新内容能够被百度收录，从而让其内容排名靠前。

  操作流程：

- 1、本地启动项目，到static/robots.txt 的线上Sitemap用#注释，本地的去掉#
- #相当于// 注释

  ```js
    // 这样
    # Sitemap: https://www.iliving.cn/sitemap.xml
    Sitemap: http://192.168.1.193:3000/sitemap.xml
  ```

- 2、然后浏览器访问 <http://192.168.1.193:3000/sitemap.xml> ，ctrl + S 保存到桌面，将这个sitemap.xml文件发给后端
- 3、后端收到后把老的xml删掉，把新的放进去

- 4、此时前端的static/robots.txt下的设置需要更改回来
      谨记：如果没改回来到时上线报错是很难定位的，因为nuxt给的报错信息非常的匮乏，你只能从自己回想刚刚更改了哪些。

   ```js
    // 这样
    Sitemap: https://www.iliving.cn/sitemap.xml
    # Sitemap: http://192.168.1.193:3000/sitemap.xml
   ```

## 开发规范

#### 1、div布局容器

由于开发固定版心是1200px，所以页面的主体内容盒子需要设置，保证页面所有内容都在中间，这个宽是根据原型图的，如果原型图中的某一块区域的宽度是占满整个宽度(例如某些大图)，这种直接在外面创建一个新div，该div跟container_box属于同级的。

```css
.container_box {
    max-width: 1200px;
    margin: 0 auto;
    position: relative;
    overflow-x: hidden;
}
```

#### 2、样式模块化

例如 home.vue

```less
<style scoped lang='less'>
@import "~/assets/css/home/index.less";
</style>
```

开发的css代码统一存放到 assets/css/XX/XX.less ，文件名根据不同页面进行命名。

#### 3、git  commit -m '标识：做了什么'

目的：方便回滚

```js
feat   新增    原单词是feature
chore  修改
merge  合并或解决冲突
style  优化样式

示例：git commit -m 'feat:首页新增了分页功能'

```

注：修改代码先切到子分支去修改，不要直接在main分支去更改，main主分支是两个子分支都没问题的情况下最终才合上来的，否则都不用创建子分支所有人都在一个分支开发，到时出问题到处甩锅和一大堆问题很难排查。

#### 4、调用接口的方式

##### 1、asyncData概念解释

首屏渲染(页面初次渲染)的数据接口必须写在asyncData中调用，asyncData它是在组件渲染之前做的事情，此时的数据才能被SEO爬虫捕捉到，再通过服务端渲染最后才正式从到组件渲染。而纯Vue(非nuxt)直接写在 created 或 mounted 就无法检测到SEO的各种属性，具体看nuxt的runtime执行原理。

原理参考链接：<https://cloud.tencent.com/developer/news/221008>

简言之：需要首次渲染前展示的数据接口统一写到asyncData钩子函数中，asyncData跟methods同级。

##### 2、需要首屏渲染的接口调用方式

进入页面就要展示的数据接口

````js
/* 获取后需要将数据返回(return)给组件，数据才能在组件中进行渲染
 这里return出来的数据不需要在data下初始就能直接使用
 参数说明：
  1、参数是一个context上下文对象，可以解构获取其中的根实例app,类似vue的this,query是接收上个页面传来的参数
  2、这里只讲了两个参数，实际上还有很多，感兴趣看nuxt文档  https://www.nuxtjs.cn/api/context
 实例说明：
  1、app是asyncData内的this，只是它的名字不是this
  2、methods里的函数this跟vue一样，不仅可以获取到vue自身的实例this,还能通过this获取到asyncData钩子内return出      来的数据
  接口参数依旧是params，基本跟vue一样，唯一不同的是params的外层需要有一个对象包裹它
  
*/
async asyncData({ app,query }){    
    
    // 不需要传参获取接口的
    const resBanner = await app.$api.home.getData("getBannerList");  
    if(banner.code == 200) {} else{}  // 每个异步请求的状态判断都跟平常调接口一样
    
    // 需要接参数调接口获取数据的
    let params = { id: query.id, ... }
    const resProduct = await app.$api.home.getData("getProductList", {params});  
    
    // ....n个接口        
    return {        
        bannerList: resBanner.list,
        prodList: resProduct.list
        // ....n个返回参数    
        }
  }
````

##### 3、首屏渲染后还想动态更改数据（通过事件更改数据的）

在以往的Vue methods钩子中进行编写即可

````js
// 示例  
methods(){
    getDetailId() {
        const res = await this.$api.home.getData("getDetail", {params});
         // 假设当前页面首屏渲染asyncData调用了bannerList接口，那么methods是可以获取到它的数据的，
        // 前提是只要asyncData有将数据return出来，这就是上面实例说明第2点讲的
        console.log(this.bannerList,'轮播的接口数据')
    }
 
}
````

##### 4、api统一管理在 api/apilist/home.js  下，以及调用实例模板写的非常详细

#### 5、如何传参

##### 1、标签传参

````js
// 三种方式任取其一
<nuxt-link :to="{ path: 'pressCenter/article?id=' + item.id }" >测试</nuxt-link>
<nuxt-link :to="{name: 'basicinfo', query: {name: 'qwe', age: 23}}">测试</nuxt-link>
<a :href="'/basicinfo?name=qwe&age=23'">测试</a>
````

##### 2、动态传参

````js
this.$router.push({path: '/branding', query: {name: 'qwe', age: 23}})
````

##### 3、接收参数

````js
// 跟methods同级的asyncData方法中获取上个组件的参数, 再用return把数据返回出去,页面才可以使用

// 示例：
methods:{}
async asyncData({ query }){
  let params = { id: query.id, ... }
    const res = await this.$api.home.getData("getProductList", {params});
     console.log(res)
     
 return {
     bannerList: res.list  // 对应接口返回的名称,这里的res.list只是举例
    }
}
````

如果还是有疑惑参考此链接：<https://blog.csdn.net/weixin_47534468/article/details/123675161>

#### 6、本地服务器和线上服务器地址的更改位置

##### 1、package.json

在package.json中的scripts进行修改，BASE_URL写后端的接口，本地接口的修改dev，线上和打包上线的改build和start

注：地址不需要带引号 " " or ''。

````js
  "scripts": {
    "dev": "cross-env BASE_URL=http://192.168.1.159:8082 NODE_ENV=dev nuxt ",
    "build:prod": "cross-env BASE_URL=http://192.168.1.159:8082 NODE_ENV=prod nuxt build",
    "start": "cross-env BASE_URL=http://192.168.1.193:8081 NODE_ENV=prod nuxt",
  },

````

##### 2、env.js  

这个公共文件的地址也需要更改，dev本地，prod线上

````js
dev: {
    NODE_ENV: "dev",
    BASE_URL: "http://192.168.1.159:8082", 
  },
  prod: {
    NODE_ENV: "prod",
    BASE_URL: "http://192.168.1.193:8081",
  },

````

#### 7、文件中如何使用公共服务器前缀  

 process.env.BASE_URL这个自动对应当前项目环境中的服务器(dev or prod)，无需引入env.js文件

```js
data() {
    return {
      filPath: process.env.BASE_URL, 
    };
  },
     
    首屏需要渲染图片的在asyncData中初始filePath图片地址前缀变量，然后return出去，无需在data(){}钩子中声明变量
  // let filPath = process.env.BASE_URL"; // 线上
    let filPath = "http://192.168.1.159" + "/dev-api"; // 线上


```

#### 8、 **那assets文件夹与static文件夹有何区别?**

共同点是都是存放静态资源的文件夹，
不同点是assets文件夹下是页面和组件中用到的静态资源，比如公共样式文件，字体图标文件，图片等。放在assets中的文件会进行压缩体积、代码格式化，压缩后会放置在static中一同上传服务器。因此建议样式文件放在assets中进行打包，引入的第三方文件放到static中，因为引入的文件已经做过打包处理。使用assets下面的资源，在js中使用的话，路径要经过webpack中file-loader编译，路径不能直接写。static中的文件，不会经过编译。项目在经过打包后，会生成dist文件夹，static中的文件只是复制一遍而已。
简单说就是，assets文件夹会经过webpack编译再打包而static文件夹不会经过编译就直接打包。所谓打包简单点可以理解为压缩体积，代码格式化。而压缩后的静态资源文件最终也都会放置在static文件中跟着index.html一同上传至服务器。
如果把图片放在assets与static中，html页面可以使用；但在动态绑定中，assets路径的图片会加载失败，因为webpack使用的是commenJS规范，必须使用require才可以使用。

#### 9、代码优化

给每个v-for 遍历的外层容器添加一层v-if 判断数组有值了才进行遍历，由于nuxt.js在服务端渲染时的接口在控制台是看不到的，哪个接口出现问题都会导致页面出不来，此时就需要将每个接口的代码慢慢注释慢慢排查，那是相当耗时而且不容易找的。用了v-if之后，页面哪部分内容没展示就可以快速定位问题的根源所在，节省很多时间，所以这是一种非常优秀的思想，为后续的bug排查提供很大的帮助。

#### 10、注意事项

哪些首屏渲染时需要同时调用多个获取数据接口，例如首页，直接用promise.all异步并发调接口（一次异步调用所有需要异步调用的接口），这样网页的渲染速度就不会异步一个一个等待，那样等接口调完数据到完整的渲染页面需要等待很长的时间，首屏渲染获取少的页面则不需要promise.all，正常一个async await调用即可。 还有一点：使用promise.all时调用接口的顺序要从页面自上而下的调用排列，如果顺序不对页面数据会出现功能丢失等异常，经验教训。

### 11、Request path contains unescaped characters on axios get request

如果当前页面需要在首屏渲染时 根据url中的获取中文参数拿去接口传参出现上面的报错，此时需要将中文参数转化为url支持的url格式  用encodeURI，这样接口才能辨识，传参就没问题了

```js
示例：
resCateName = await app.$api.home.getCateData(
    "getProdCateList",
    encodeURI(query.categoryName)
  );

```

#### 11、后言

这个项目其实还做了诸多优化，cdn、element ui按需导入、并发统一调异步接口、webpack各种打包压缩，gzip、br压缩、后端nginx也做了gzip文本图片压缩，性能优化相关的都用了一部分，无论是代码、接口、网页渲染、各种压缩等。最终的目标是SEO排名靠前、同时网站各种性能比较强，这些都是一个优秀网站不可或缺的重要组成部分。

Author：HaushoLin

2022/11/24 13:33