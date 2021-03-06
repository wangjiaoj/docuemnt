# 兼容低版本浏览器

## 一、基础概念

1. Babel是一个新语法转旧语法的平台，它只对synax进行转义，对于api需要使用其对应的插件进行转化。新的API，比如 Iterator、Generator、Set、Maps、Proxy、Reflect、Symbol、Promise 等全局对象，以及一些定义在全局对象上的方法（比如 Object.assign ）都不会转码。

2. 对于API的转换又分为两部分，一个是全局API例如Promise，Set，Map还有静态方法Object.assign，另一个是实例方法例如Array.prototype.includes。对于实例方法core-js@2是转换不了的，只有core-js@3才会转换。
   
3. `@babel/polyfill`和`@babel/plugin-transform-runtime`都可以实现api转换。但是前者会污染全局，后者不会。如果不是编写插件或库，可以使用@babel/polyfill，否则请使用`@babel/plugin-transform-runtime`。

babel官方文档：[@babel/polyfill](https://www.babeljs.cn/docs/babel-polyfill)
翻译文档参考[翻译文档参考](https://blog.windstone.cc/es6/babel/)
 `@babel/polyfill`和`@babel/plugin-transform-runtime`两种区别一个是会修改全局变量
 >As of Babel 7.4.0, this package has been deprecated in favor of directly including core-js/stable (to polyfill ECMAScript features) and regenerator-runtime/runtime (needed to use transpiled generator functions):

babel6到babel7的升级是具有破坏性的，主要总结下polyfill的用法和在babel6和babel7中的不同。



#### 1.1 @babel/preset-env

基本的语法转换，需要添加预设 @babel/preset-env。
[babel7中 preset-env 完全使用](https://www.cnblogs.com/htoooth/p/9877524.html)
[官方preset-env配置项](https://www.babeljs.cn/docs/babel-preset-env#usebuiltins)
preset-env是ES语法插件的合集，官方已经不再推荐使用preset-201x之类的包，该包可以通过配置自动兼容代码，包括自动引入polyfill垫片处理新的API（例如：Promise,Generator,Symbol等）以及 实例方法（例如Array.prototype.includes等）。

 也就是说preset-env是基础语法转换，需要配合`@babel/polyfill`或`@babel/plugin-transform-runtime`使用

如果preset-env是有配置项的：
 ````javascript 
 {
  "presets": [
    [
      "env",
      {
        // 这里就是配置项-比如
        "modules": false, // 对ES6的模块文件不做转化，以便使用tree shaking、sideEffects等
      }
    ]
  ]
}
 ````
 
可以观察出如果某个preset需要配置可以将字符串换成一个数组，第一项是preset的name，第二项就是options。plugin同preset。

### 1.2 babel7配套升级要求

babel 7.0 要求 nodejs >= 6。

babel：
* babel-core
* babel-cli
* babel-loader

其中
* babel-loader 8.x对应babel-core 7.x
* babel-loader 7.x对应babel-core 6.x

babel7升级之后需要升级配套loader等
* babel-eslint v9: https://github.com/babel/babel-eslint/releases/tag/v9.0.0
* babel-loader v8: https://github.com/babel/babel-loader/releases/tag/v8.0.0
* gulp-babel v8: https://github.com/babel/gulp-babel/releases/tag/v8.0.0
* rollup-plugin-babel v4: https://github.com/rollup/rollup-plugin-babel/tag/v4.0.3

另外babel7.4.0也是一个分水岭，百度的估计都是7.4.0之前的配置方案(个别插件废弃 + 配置繁琐)，需要注意package.json的babel版本。 

## 二、babel7
参考文章：
[@babel/polyfill和@babel/plugin-transform-runtime](https://blog.csdn.net/m0_37613019/article/details/108226550)
[结合Babel 7.4.0 谈一下Babel-runtime 和 Babel-polyfill](https://juejin.im/post/6844903869353295879)

babel7.4.0+
7.4 开始，就不推荐使用 @babel/polyfill 了
[babel7.4之后babel配置](https://blog.csdn.net/qq_21567385/article/details/107104592?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.compare&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.compare)

### 2.1关于babel7.4.0文档中描述的不推荐使用polyfill的问题。
其实polyfill本身就是stable版本的core-js和regenerator-runtime的合集，即import @babel/polyfill等同于：
>import 'core-js/stable';
>import 'regenerator-runtime/runtime';
所以在针对Babel >= 7.4.0 的情况，我们需要安装 core-js 替代 babel-polyfill,而 regenerator-runtime 会在我们安装 @babel/runtime 时自动安装，所以不必单独安装了。


### 2.2 @babel/plugin-transform-runtime
>npm install --save-dev @babel/plugin-transform-runtime
>npm install --save @babel/runtime

 
在babel7中，原先的插件babel-plugin-transform-runtime也做了修改 -> @babel/plugin-transform-runtime。并且在功能上也变强大了。

移除了polyfill的配置项添，加了corejs配置项。

@babel/plugin-transform-runtime的默认配置如下：

 ````javascript
 {
  "plugins": [
    [
      "@babel/plugin-transform-runtime",
      {
        "absoluteRuntime": false,
        // 	配置corejs为3，需要预先安装@babel/runtime-corejs3
      	// 配置corejs为2，需要预先安装@babel/runtime-corejs2
      	// 配置corejs为false，需要预先安装@babel/runtime
        "corejs": false,
        "helpers": true,
        "regenerator": true,
        "useESModules": false,
      }
    ]
  ]
}
````
1. 对于配置项 corejs: false | 2 | 3。


false: 依赖 @babel/runtime，是默认选项，默认不启用对core-js的polyfill处理的，认为需要填充的API已被填充，所以不对API作polyfill（对于@babel/runtime的依赖是提取的helper的依赖）。
2: 依赖@babel/runtime-corejs2
3: 依赖@babel/runtime-corejs3

corejs2和corejs3区别。
corejs2只转换全局变量(Object)和静态方法(Object.assign)，并不转换原型上的方法(Array.prototype.includes)，corejs3会转换原型上的方法。

如果设置成false，npm run build之后，会发现Promise方法并没有被转换。

2. polyfill和useBuiltIns两个配置项
This option was removed in v7 by just making it the default.


3. 小结：
@babel/plugin-transform-runtime主要有三个作用：
- 当使用 generators/async的时候自动引入 @babel/runtime/regenerator。
- 为新特性的API添加实现。
- 提取每个模块内联的helper们问引用。

### 2.3 @babel/polyfill + core-js@3
>npm install --save @babel/polyfill

Babel7 的 presets对babel-polyfill做了处理，新增"useBuiltIns": "usage"，这样只会加载代码中用到的部分，完美的按需加载，配置browserlist和@babel/preset-env的useBuiltsIns属性。

 ````javascript
{
  "presets": [
    [
    // babel的编译预设 
      "@babel/env",
      {
          //针对项目指定生成适应环境的代码。如果不进行配置，babel会转义所有ES6+进行环境适配，十分不推荐该用法。
           "targets": {},
        // 该选项用来配合@babel/polyfill进行使用，针对Babel > 7.4.0, 官方不再推荐使用该库，请选择core-js，根据安装core-js版本在corejs选项填写数字2或3。  usage-仅引入使用到的语法转换
        "useBuiltIns": "usage",
        "corejs": { "version": 3, "proposals": true }
      }
    ]
  ]
}
````
1. useBuiltIns
分为三种，即 entry, usage 和 false。 usage 就是按需导入, entry 则需要手动导入，很不人性化，所以我们就用 usage 即可。

原文件里需不需要手动写导入
官方文档上来就说让我们手动导入一遍：

import "core-js/stable";
import "regenerator-runtime/runtime";


其实是不需要的，会自动帮我们导入，那怎么确认这两个工具（ core-js 和 regenerator-runtime ）生没生效呢：

1. 如果多导入了，会报错，取消导入即可
2. 如果少导入了 regenerator-runtime ，会发现文件开头部分没有导入 require(“regenerator-runtime/runtime”);
3. 如果少导入了 core-js ，会发现文件开头没有导入 object 和 promise 等方法
按照这个方法就可以判断这两个工具生效没生效。

总结
可见现在官方还没彻底放弃 @babel/polyfill ，我们直接使用外置的 core-js@3 依赖，这样就完美解决问题了。


#### 2.4小结：
Babel < 7.4.0
开发类库, 选择 @babel/runtime
内部项目，@babel/polyfill
Babel >= 7.4.0，啥也不说，直接上 @babel/runtime吧，因为你要的全都有啊



## 四、babel6
 >If you are looking for something that won't modify globals to be used in a tool/library, checkout the transform-runtime plugin. This means you won't be able to use the instance methods mentioned above like Array.prototype.includes.

### 4.1 直接引入（影响全局，一劳永逸）

在入口文件中 import 'babel-polyfill' / require('babel-polyfill')。使用webpack的话也可以在entry中添加 entry: ['babel-polyfill', 'src/index.js']。

优点：
一次引入，全局使用。
会转换实例方法和静态方法，比较全面。

缺点：
影响全局作用域。
打出来的包比较大，不管用没用到都打进去了。

使用场景：
开发业务项目，比较全面，不会漏了从而出问题，比如Object.assign这样的方法在ios8上面还是需要polyfill的。

 

 
 ````javascript
 const presets = [
  [
    "@babel/env",
    {
      targets: {
        edge: "17",
        firefox: "60",
        chrome: "67",
        safari: "11.1",
      },
    },
  ],
];

module.exports = { presets };
 ````


 

### 4.2.  在babel-runtime中单独引入
为了不污染全局对象和内置的对象原型，但是又想体验使用新鲜语法的快感。就可以配合使用babel-runtime和babel-plugin-transform-runtime
1. `npm install --save-dev babel-plugin-transform-runtime`
2. `npm install --save babel-runtime`

和直接在入口引入polyfill不同，该插件引入的polyfill是模块私有的。
对于需要的polyfill需要手动引入，import Promise from 'babel-runtime/core-js/promise'

优点：
该模块私有，不会影响到全局作用域。
打出来的包因为按需引入包不会很大。

缺点：
因为不影响全局作用域，所以不会转实例和静态方法这样的API。
手动引入所需，搞不好会漏掉。

使用场景：
开发库，框架之类可以使用，因为别人用你的东西然后不知情的情况下你改了别人的全局环境，然后出错了就尴尬了。

### 4.3. 使用babel-plugin-transform-runtime按需引入

这个插件可不简单，有好几个功能：

和插件babel-runtime一样引入的polyfill是私有的，不会影响全局作用域。并且是自动按需引入需要的polyfill，不需要想babel-runtime一样一个一个手动引入。
可以提取语法转换时候每个模块都生成的各种helper，成一个引用。
自动转换generators/async。
优点：
该模块私有，不会影响到全局作用域。
打出来的包因为按需引入包不会很大。
自动按需引入，不需要手动，防止遗漏。
提取helper，大大减少冗余代码。

缺点：
因为不影响全局作用域，所以不会转实例和静态方法这样的API。

使用场景：
同babel-runtime。

注：该插件依赖babel-runtime。

按需加载

 ````javascript
{
  "presets": [
    ["env", {
      "modules": false,
      "targets": {
        "browsers": ["> 1%", "last 2 versions", "not ie <= 8"]
      }
    }],
    "stage-2"
  ],
  "plugins": ["transform-vue-jsx", "transform-runtime"],
  "env": {
    "test": {
      "presets": ["env", "stage-2"],
      "plugins": ["transform-vue-jsx", "transform-es2015-modules-commonjs", "dynamic-import-node"]
    }
  }
}
 ````


 [API转换](https://www.cnblogs.com/zhishaofei/p/10058132.html)


## 4.4
在babel-preset-env中设置配置项-----不是7才有这个配置项的吗？疑惑
 ````javascript
{
   "presets": [
   	[
   		"env", 
   		{
   			"useBuiltIns": boolean
   		}
   	]
   ]
}
 ````
useBuiltIns 选项是为了分割入口的 import 'babel-polyfill' / require(babel-polyfill)成按环境引入polyfill。该方式同第一中引入polyfill的方式，但是会按照配置的环境去按需引入，稍微好点。
 

小结：
1. babel6的核心有 babel-core babel-cli babel-node babel-register babel-polyfill，这些在babel7会有所修改。
2. polyfill 是依赖core-js的
3. babel7.4放弃了@babel/polyfill直接依赖core-js@2或者3。

## 五、样式
兼容不同浏览器，进行样式补全
*  Autoprefixer
* postcss
 Autoprefixer是一个后处理程序，不象Sass以及Stylus之类的预处理器。它适用于普通的CSS，可以实现css3代码自动补全。也可以轻松跟Sass，LESS及Stylus集成，在CSS编译前或编译后运行。

 webpack.config.js

 ````javascript
 {
        test: /\.css$/,
        use: [
         {
            loader: 'style-loader'
          },
          {
            loader: 'css-loader',
            options: {
              modules: true,
              importLoaders: 1,
              minimize: true,
              localIdentName: '[local]_[hash:base64:5]'
            }
          },
          {
            loader: 'postcss-loader'
          }
        }
             
 ````

postcss.config.js

````javascript
module.exports = {
    plugins: [
        require('autoprefixer')({
            "browsers": [
                "defaults",
                "not ie < 11",
                "last 2 versions",
                "> 1%",
                "iOS 7",
                "last 3 iOS versions"
            ]
        })
    ]
};
 ````





 
## 六. 更低版本的IE(8)兼容

### 6.1、ECMAScript 5浏览器兼容性

ECMAScript是一种由Ecma国际在标准ECMA-262中定义的脚本语言规范。

ECMA-262第5版于2009年12月发布，新增 "严格模式（strict mode）"，一个子集用作提供更彻底的错误检查,以避免结构出错。

除此而外，还有 Function.prototype.bind、数组方法 indexOf、forEach、map 和 filter，对象方法 defineProperty、create 和 keys等新特性。

浏览器支持情况如下所示：

* IE8
不支持。但支持 Object.defineProperty、Object.getOwnPropertyDescriptor 和 JSON解析等功能。
* IE9
部分支持。不支持严格模式（Strict mode）
* IE10+
支持。发布时间：2012年。
* Firefox 21
支持。发布时间：2013年。
* Chrome 23+
支持。发布时间：2012年。
* Safari 6+
支持。发布时间：2012年。
* IOS Safari 6+
支持。发布时间：2013年。
* Android Browser 4.1-4.3
部分支持。存在parseInt()函数问题
* Android Browser 4.4+
支持。发布时间：2013年。
* Chrome for Android 79+
支持。发布时间：2019年。

### 6.2 解决方案
* es5-shim/es5-sham
在模拟不兼容 es5 语法的浏览器，典型的 ie 6/7/8 浏览器
es5-shim和es5-sham的引用是必要的，因为它是解决通用性的es3环境下es5 API的缺失问题，就像babel-polyfill一样，Object.defineProperty是其中的一个API
* es3ify-loader
ES5->ES3
解决es3的保留字在es3环境下的正确使用， 它主要做的事情就是对于一些保留字的使用做了es3兼容,以及一些额外的处理，比如去除数组尾部的多余逗号：
* "transform-es3-property-literals",
  保证在对象属性中的保留字正确
*  "transform-es3-member-expression-literals"
 保证在对象属性访问中的保留字正确
* console-polyfill
* 不要在代码中用Object.defineProperty设置访问器属性，若第三方包中有，找到，改之

UglifyJS本来就已经提供了对IE浏览器的支持，不需要额外引入es3ify-loader。webpack默认的UglifyJS配置不支持ie8，需要手动配下


## 七、其他常见babel插件
组件按需加载之babel-plugin-import
下图示例import
 ````javascript 
    "plugins": [
        "transform-runtime",
        "babel-plugin-transform-regenerator",
        "transform-vue-jsx",
        [
          "import",
          {
            "libraryName": "vant",
            "libraryDirectory": "es",
            "style": true
          }
        ],
        "syntax-dynamic-import"
    ]
}
````

## 八、小结
[康一康谁说的对](https://zhuanlan.zhihu.com/p/147083132)
作者告诉你useBuildInts 和babel/plugin-transform-runtime不能混用. 参看
https://github.com/babel/babel/issues/10271#issuecomment-528379505

issue10008 说得很清楚了呀，而且@babel/plugin-transform-runtime主要是给开发类库时用的，不是用来解决preset的问题的。 transform runtime环境无关，不会因为你的页面的目标浏览器动态调整polyfill的内容；而useBuildInts会。两者是不同目的的东西

还有这 https://github.com/babel/babel/issues/10133#issuecomment-506249922

babel/plugin-transform-runtime是环境无关的

如果@babel/plugin-transform-runtime配置了corejs:3，preset-env的useBuiltIns就不会生效，babel/plugin-transform-runtime会打包所有浏览器的polyfill