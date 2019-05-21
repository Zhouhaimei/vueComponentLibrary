# bwt-uikit部署流程以及开发指南
说明文档地址：https://zhouhaimei.github.io/tech/vueComponentLibrary.html
仿照elemnt-ui，实现按需加载，文档示例说明，以及单元测试
- 目录结构

```js
.
├── README.md
├── build
│   ├── build.js
│   ├── check-versions.js
│   ├── logo.png
│   ├── strip-tags.js
│   ├── utils.js
│   ├── vue-loader.conf.js
│   ├── webpack.base.conf.js
│   ├── webpack.dev.conf.js // 开发配置，启动examples示例
│   ├── webpack.prod.conf.js // 生产配置 打包输出lib目录供其他项目引用
│   ├── webpack.prod.examples.conf.js // 示例打包，可部署的文件
│   └── webpack.test.conf.js //测试配置
├── components.json
├── config
│   ├── dev.env.js
│   ├── index.js
│   ├── prod.env.js
│   └── test.env.js
├── examples //示例
│   ├── App.vue
│   ├── assets
│   │   └── less
│   ├── components
│   │   ├── demoBlock.vue
│   │   ├── header.vue
│   │   └── menu.vue
│   ├── docs //真正的说明文档
│   │   ├── button.md
│   │   ├── input.md
│   │   ├── install.md
│   │   └── quikeStart.md
│   ├── index.html
│   ├── main.js
│   └── route.js
├── dist // example build输出目录
├── lib // 发布时打包输出目录
│   ├── button-model.js
│   ├── button-model.js.map
│   ├── index.js
│   ├── index.js.map
│   ├── input.js
│   ├── input.js.map
│   └── theme
│       ├── button-model.css
│       ├── button-model.css.map
│       ├── index.css
│       ├── index.css.map
│       ├── input.css
│       └── input.css.map
├── package.json
├── src // 组件源码
│   ├── components
│   │   ├── HelloWorld.vue
│   │   ├── buttonModel
│   │   └── input
│   └── index.js
├── static
└── test // 单元测试
    ├── e2e
    │   ├── custom-assertions
    │   ├── nightwatch.conf.js
    │   ├── runner.js
    │   └── specs
    └── unit
        ├── coverage
        ├── index.js
        ├── karma.conf.js
        └── specs
```
## 原理介绍
- element-ui按需加载的原理
    > 使用babel-plugin-component实现按需引入、打包。
    将webpack配置成多入口，保证最终打包的目录结构符合babel-plugin-component插件的要求.

- [babel-plugin-component](https://www.npmjs.com/package/babel-plugin-component) 
使用示例
    
    Converts
    ```
    import { Button } from 'components'
    ```
    to
    ```
    var button = require('components/lib/button')
    require('components/lib/button/style.css')
    ```
- 以项目中实际使用bwt-uikit为例，按需引入

    首先，安装 babel-plugin-component：
    
    ```bash
    npm install babel-plugin-component -D
    ```
    
    然后，将 .babelrc 修改为：
    
    ```json
    {
      "presets": [["es2015", { "modules": false }]],
      "plugins": [
        [
          "component",
          {
            "libraryName": "bwt-uikit",
            "styleLibrary":{
              "name":"theme",
              "base":false
            }
          }
        ]
      ]
    }
    ```
    
     项目入口文件
     
    ```
    import Vue from 'vue';
    import { Input } from 'bwt-uikit';
    import App from './App.vue';
    
    Vue.component(Input.name, Input);
    /* 或写为
     * Vue.use(Input)
     */
    
    new Vue({
      el: '#app',
      render: h => h(App)
    });
    ```
    
    实际上Converts
    
    ```
    import { Input } from 'bwt-uikit';
    ```
    
    to
    
    ```
    var Input = require('bwt-uikit/lib/input')
    require('bwt-uikit/lib/theme/input.css')
    ```
    
    为了生效，bwt-uikit 包的目录结构如下：（babel-plugin-component插件推荐的目录方案之一）
    
    ```js
    ├── lib
    │   ├── button-model.js
    │   ├── button-model.js.map
    │   ├── index.js
    │   ├── index.js.map
    │   ├── input.js // 1.通过配置自动生成
    │   ├── input.js.map
    │   └── theme // 'styleLibraryName'
    │       ├── button-model.css
    │       ├── button-model.css.map
    │       ├── index.css //required
    │       ├── index.css.map
    │       ├── input.css //2.通过配置自动生成
    │       └── input.css.map
    ```
## 实现上述目标-bwt-uikit的打包发布配置
- `build/webpack.prod.conf.js` 配置多入口打包

```
const components = require('../components.json')
const entrys = {}
Object.keys(components).forEach(item => {
  entrys[item] = components[item]
})
```
- `/components.json`
```
{
  "input": "./src/components/input",
  "button-model": "./src/components/buttonModel",
  "index": "./src/index.js"
}

```
- `build/webpack.prod.conf.js` 配置输出目录
```
const config = require('../config')
//...
 output: {
    path: config.build.assetsRoot,
    filename: '[name].js',
    library: 'VueDropUpload',
    libraryTarget: 'umd'
  },
 //...
```
- `config/index.js`

```
 build: {
    // Paths
    assetsRoot: path.resolve(__dirname, '../lib'),
  }
```
到此完成bwt-uikit的打包发布配置
## 示例文档说明开发

```
├── examples
│   ├── App.vue
│   ├── assets //静态资源依赖
│   │   └── less
│   ├── components //vue组件
│   │   ├── demoBlock.vue
│   │   ├── header.vue
│   │   └── menu.vue
│   ├── docs //存放.md文件，说明文档
│   │   ├── button.md
│   │   ├── input.md
│   │   ├── install.md
│   │   └── quikeStart.md
│   ├── index.html
│   ├── main.js //examples 部署生产环境的打包入口，在这里引入项目的组件、第三方依赖：element-ui、路由配置等
│   └── route.js
```
- `build/webpack.prod.examples.conf.js` 打包配置输出目录
```
// 判断参数是打包examples,设置output出口为dist
if (process.argv[2] == 'examples') {
  config.build.assetsRoot = path.resolve(__dirname, '../dist')
}
//。。。
 entry: {
    // 设置打包example的入口
    app: './examples/main.js'
  },
//。。。
//...
// 设置html模版为./examples/index.html
    new HtmlWebpackPlugin({
      template: './examples/index.html',
    }),
//...
```
- 安装开发依赖
```
npm i highlight -D //安装语法高亮
npm i markdown-it markdown-it-anchor markdown-it-container -D // 安装markdown相关依赖
npm i vue-markdown-loader -D //安装vue-markdown-loader，解析.md文件为.vue文件
```

- examples开发配置`build/webpack.base.conf.js`

```
// 从HTML字符串中删除特定标记。参考 https://github.com/jonschlinkert/strip-tags
const striptags = require('./strip-tags')
//markdown-it 是一个辅助解析 markdown 的库，可以完成从 # test 到 <h1>test</h1> 的转换。
const md = require('markdown-it')()
// ...
{
        test: /\.md$/,
        loader: 'vue-markdown-loader',
        options: {
          use: [
          // 参考markdown-it-container examples
            [require('markdown-it-container'), 'demo', {
              validate: function(params) {
                return params.trim().match(/^demo\s*(.*)$/)
              },
              render: function(tokens, idx) {
                var m = tokens[idx].info.trim().match(/^demo\s*(.*)$/)
                if (tokens[idx].nesting === 1) {
                  var description = (m && m.length > 1) ? m[1] : ''
                  var content = tokens[idx + 1].content
                  var html = convert(striptags.strip(content, ['script', 'style'])).replace(/(<[^>]*)=""(?=.*>)/g, '$1')
                  var script = striptags.fetch(content, 'script')
                  var style = striptags.fetch(content, 'style')
                  var jsfiddle = { html: html, script: script, style: style }
                  var descriptionHTML = description
                    ? md.render(description)
                    : ''

                  jsfiddle = md.utils.escapeHtml(JSON.stringify(jsfiddle))

                  return `<demo-block class="demo-box" :jsfiddle="${jsfiddle}">
                            <div class="source" slot="source">${html}</div>
                            ${descriptionHTML}
                            <div class="highlight" slot="highlight">`
                }
                return '</div></demo-block>\n'
              }
            }],
            [require('markdown-it-container'), 'tip'],
            [require('markdown-it-container'), 'warning']
          ],
          preprocess: function(MarkdownIt, source) {
            MarkdownIt.renderer.rules.table_open = function() {
              return '<table class="table">';
            };
            MarkdownIt.renderer.rules.fence = wrap(MarkdownIt.renderer.rules.fence)
            return source;
          }
        }
      },
//...

```
- [markdown-it-container](https://cnpmjs.org/package/markdown-it-container/)简单使用
    
    Converts
    ```
    ::: warning
    *here be dragons*
    :::
    ```
    无配置render默认转义to 
    
    ```
    <div class="warning">
    <em>here be dragons</em>
    </div>
    ```
    API
    --
    
    ```
    var md = require('markdown-it')()
                .use(require('markdown-it-container'), name [, options]);
    ```
    
    Params:
    
    - name - container name (mandatory)
    - options:
        - validate - optional, function to validate tail after opening marker, should return true on success.
        - render - optional, renderer function for opening/closing tokens.
        - marker - optional (:), character to use in delimiter.
    
    Example
    --
    
    ```
    var md = require('markdown-it')();
    
    md.use(require('markdown-it-container'), 'spoiler', {
    
      validate: function(params) {
        return params.trim().match(/^spoiler\s+(.*)$/);
      },
    
      render: function (tokens, idx) {
        var m = tokens[idx].info.trim().match(/^spoiler\s+(.*)$/);
    
        if (tokens[idx].nesting === 1) {
          // opening tag
          return '<details><summary>' + m[1] + '</summary>\n';
    
        } else {
          // closing tag
          return '</details>\n';
        }
      }
    });
    
    console.log(md.render('::: spoiler click me\n*content*\n:::\n'));
    
    // Output:
    //
    // <details><summary>click me</summary>
    // <p><em>content</em></p>
    // </details>
    ```
- 设置路由`examples/route.js`
```
import Install from './docs/install.md'
//...
const routes = [
  {
    path: '/',
    component: Install,
    name: 'default'
  },
 //...
]

export default routes

```
- 设置入口`examples/main.js`
- 
```

// 引入组件
import bwtUikit from '../src'
Vue.use(bwtUikit)
// 引入demo-block
import DemoBlock from './components/demoBlock'
Vue.component('demo-block', DemoBlock)

// 引入路由
import routes from './route'
//...

```
## 单元测试

构建建议使用vue-cli
- karma
> 是一个测试运行器，为开发者提供高效的测试环境，主要作用是将项目运行在各种主流Web浏览器进行测试。
关于karma的学习，建议看官方文档。
组件库项目是通过vue-cli搭建的，项目生成时karma相关配置就已经设置好了，关于karma，可以先作为了解即可。

- mocha
> mocha是一个测试框架，兼容多种断言库，mocha的学习可以看阮一峰老师的测试框架 Mocha 实例教程进行了解。

- chai
> chai是一个测试断言库，所谓断言，就是对组件做一些操作，并预言产生的结果。如果测试结果与断言相同则测试通过。chai的学习可以参阅Chai.js断言库API中文文档

- sinon
> sinon是一个测试工具，可以使用sinon来进行模拟http等异步请求操作，作为间谍监听回调函数调用等功能来帮助我们更轻松实现测试。sinon学习参阅：sinon入门,关于模拟http请求：利用SinonJS测试 AJAX 请求例子

- @vue/test-utils
> @vue/test-utils是vue官方推荐的vue测试工具，使用这个工具我们可以让我们更方便的测试vue项目。官方文档：vue-test-utils

### 注意事项
- 1、karma.conf.js中的browsers参数需要改成Chrome,并安装chrome相关依赖;
- 2、要测试的vue组件有依赖其他第三方插件，需要在@vue/test-utils中引入localVue，并将第三方插件注册到localVue中，mount挂载组件生成wrapper时，将localVue作为参数传递；
- 3、要测试的组件引入element-ui，除了要在localVue中注册外，还需引入@vue/test-utils的config,并进行配置：

    config.stubs.transition = false  
    config.stubs['transition-group'] = false
- 4、使用了element-ui的按钮等元素，绑定原生事件（比如点击事件）时，加上.native：@click.native="click"
- 5、有异步的内容，比如延时定时器，不要忘记done()，否则不会被捕获；

##  bwt-uikit 包本地开发测试建议

压缩包为.tgz文件再安装

```bash
~/workspace/package-name $ npm pack
~/workspace/package-name $ cp package-name-0.0.0.tgz ~
~/workspace/some-application $ npm install ~/package-name-0.0.0.tgz
```

## 测试按需加载

全部引入

```
                                                  Asset       Size  Chunks                    Chunk Names
                                          vendor.dll.js     126 kB          [emitted]         
                    static/js/0.c17789e560b74491b3a6.js  905 bytes       0  [emitted]         
    static/css/app.8d098f87028019bcb7a17e08d666e71a.css    9.78 kB       1  [emitted]         app
static/css/app.8d098f87028019bcb7a17e08d666e71a.css.map    37.2 kB          [emitted]         
                static/js/0.c17789e560b74491b3a6.js.map    5.29 kB       0  [emitted]         
              static/js/app.02e93ee1325a72c29fff.js.map    1.65 MB       1  [emitted]         app
                  static/js/app.02e93ee1325a72c29fff.js     333 kB       1  [emitted]  [big]  app
                                             index.html  721 bytes          [emitted]         
                                     static/config.json  262 bytes          [emitted]         
                                          static/rem.js    3.17 kB          [emitted]         
                                       vendor.dll.js.gz      43 kB          [emitted]         
               static/js/app.02e93ee1325a72c29fff.js.gz     115 kB          [emitted]      
```
按需加载引入
```
                                                  Asset       Size  Chunks                    Chunk Names
                                          vendor.dll.js     126 kB          [emitted]         
                    static/js/0.c17789e560b74491b3a6.js  905 bytes       0  [emitted]         
    static/css/app.14afde5c406f94fc3226f75e5561566f.css    9.67 kB       1  [emitted]         app
static/css/app.14afde5c406f94fc3226f75e5561566f.css.map      37 kB          [emitted]         
                static/js/0.c17789e560b74491b3a6.js.map    5.29 kB       0  [emitted]         
              static/js/app.3fa7cbc6ce4fa3af6e24.js.map    1.65 MB       1  [emitted]         app
                  static/js/app.3fa7cbc6ce4fa3af6e24.js     332 kB       1  [emitted]  [big]  app
                                             index.html  721 bytes          [emitted]         
                                     static/config.json  262 bytes          [emitted]         
                                          static/rem.js    3.17 kB          [emitted]         
                                       vendor.dll.js.gz      43 kB          [emitted]         
               static/js/app.3fa7cbc6ce4fa3af6e24.js.gz     114 kB          [emitted]  
```
# 参考

- https://segmentfault.com/a/1190000015884948

- https://segmentfault.com/a/1190000016342795

- https://segmentfault.com/a/1190000017970919