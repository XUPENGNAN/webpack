# webpack
## WebPack 可以看做是模块打包机它可以分析你的项目结构依赖，对JS模块依赖分析打包并对一些浏览器不能直接运行的拓展语言（Sass，TypeScript等）编译；并将其转换和打包为浏览器认识的代码供浏览器使用。在 webpack3.0 出现后，Webpack 还可以优化项目,比如 Webpack 在生产环境中有一个重要的作用就是减少 http 的请求数，它会把把多个文件入口js根据入口和依赖打包到一个bundle.js 里，这样请求数就可以减少
## Webpack属于Node的一个模块，它在Node的环境中执行，因此它可以使用Node的模块比如path,并且需要使用CommonJs模块
## 安装
### webpack通过 npm 来进行安装,如果您使用的是 webpack4 或更高版本，则还需要安装 CLI。推荐局部安装便于升级与管理不同项目的 webpack 版本。因为如果 webpack 是本地全局安装，那么npm在按照 package.json 文件下载包依赖时，会拉扯本地电脑上的 webpack 版本，不利于版本更新
```
npm install --save-dev webpack
npm install --save-dev webpack@<version>
npm install --save-dev webpack-cli
webpack -v
```
## webpack.config.js 配置文件
### 入口文件(entry):entry以哪里作为入口来建立模块的相互依赖
#### 单入口配置
```
const path = require('path');
module.exports = {
    entry:"./src/main.js",
    output:{
        path: path.resolve(__dirname, 'dist'),
        filename: 'bundle.js'
    }
}
```
#### 多入口的配置
```
module.exports = {
    entry: {
        pageOne: './src/pageOne/index.js',
        pageTwo: './src/pageTwo/index.js',
        pageThree: './src/pageThree/index.js'
  }  
};
```
### 出口文件(output):
#### output.path;目录对应一个绝对路径
#### output.filename;可以静态和动态配置
#### output.publicPath;在开发环境与生产环境中的使用是不同的；在开发环境中：是指在启用webpack-dev-server时，开发打包时生成的静态资源所在的位置。在生产环境中：指的是在打包资源时，index.html页面内引入的静态资源的路径为publicPath+的合成路径。
```
//单出口的配置
const path = require('path');
module.exports={
    mode:'development',
    //入口文件的配置项
    entry:{
        main:'./src/main.js'
    },
    //出口文件的配置项
    output:{
        //输出的路径
        path:path.resolve(__dirname,'dist'),
        //输出的文件名称
        filename:'bundle.js',
        publicPath:"./",                    //publicPath:"http://localhost:1717"
    }
}
```

```
//多出口的配置
const path = require('path');
module.exports = {
    entry: {
        pageOne: './src/pageOne/index.js',
        pageTwo: './src/pageTwo/index.js',
        pageThree: './src/pageThree/index.js'
  },
    output:{
        path: path.resolve(__dirname, 'dist'),   //多出口文件路劲
        filename: '[name].bundle.js',            //多出口文件名
        publicPath:"./",  
    }
};
```
### HtmlWebpackPlugin:生成一个html5的文件并且将webpack打包好的bunld.js出口文件自动引入html之中
#### npm install --save-dev html-webpack-plugin
#### 在使用webpack打包时会自动替换dist打包文件
```
//webpack的插件必须先require再使用
const HtmlWebpackPlugin = require('html-webpack-plugin');
plugins: [
        new HtmlWebpackPlugin({
            minify:{
                removeAttributeQuotes:true
            },
            hash:true,
            template:'./src/index.html'
        })
    ]
```

```
//多入口使用 html-webpack-plugin
const path = require('path');
const fs = require('fs');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const glob = require('glob');

let dirname = path.resolve(__dirname, "src");
let htmlPlugins = [];

//多进口文件函数
function entrys(){
    let entrysObj = {};
    let entrysname = fs.readdirSync(path.resolve(__dirname, "src"));

    for(let i = 0; i < entrysname.length; i++){
        let name = entrysname[i];
        entrysObj[name] = `./src/${name}/js/${name}.js`;

        htmlPlugins.push(new HtmlWebpackPlugin({
            minify:{
                removeAttributeQuotes:true
            },
            hash:true,
            filename: `${name}.html`,
            template: `./src/${name}/html/${name}.html`,
            //common代表公共模块，name就是html对应的同名js文件
            //这个配置将会自动在生成的html中插入你指定的js
            chunks: ['common', name]
        }));
    };
    return entrysObj;
};

module.exports = {
    entry:entrys(),
    output:{
        path:path.resolve(__dirname,"dist"),      //打包文件的产出路径
        filename:"[name].bundle.js",              //打包文件名
        publicPath:"/"                            //在生产环境是产出的静态文件的路径为publicPath+产出所在目录
                                                  //在开发环境就是开发的引用静态文件的路径
    },
    plugins:[
       ...htmlPlugins,
    ]
}
```

### 服务(webpack-dev-server)
#### 开启一个前端服务器的调试模式，此模式打包的文件生成在内存中，并且不是可见的
#### 启动命令是:webpack-dev-server --open
#### 由于浏览器有跨域的设置，我们本地调试接口时其实是跨域的那就需要一个代理进行帮助跨域：webpack-dev-server有这个跨域代理的设置，这是开发阶段调试接口的跨域问题的解决办法
#### devServer.hot;开启热替换模块（以前的热替换插件为HotModuleReplacementPlugin，在webpack3.0 以上的版本webpack-dev-server已经内置了热替换模块）
```
module.exports = {
    mode:"development",
    entry: entries(),
    output: {
        path: path.resolve(__dirname, 'dist'),
        filename: '[name].bundle.js',
    },
    devServer: {
        //告诉服务器从哪个目录中提供内容
        contentBase: path.resolve(__dirname, 'dist'),
        //服务器的IP地址，可以使用IP也可以使用localhost
        //当设置为0.0.0.0后将项目部署在服务器上启动测试的dev-server也可以进行访问
        host: '0.0.0.0',
        //服务端压缩是否开启
        compress: true,
        //配置服务端口号
        port: 3000,
        disableHostCheck:true,
        hot: true
    }
};
```

```
proxy:{
        "/api": {
            target: 'http://localhost:8888',
            pathRewrite: {'^/api' : ''},
            changeOrigin: true,
        }
    },
```

### babel转译es6语法
#### 将浏览器不支持的es6语法转译为es5语法
#### npm install babel-loader@8.0.0-beta.0 @babel/core @babel/preset-env webpack --save-dev
```
 {
    test: /\.js$/,
    exclude: /(node_modules)/,
    use: {
        loader: "babel-loader",
        options: {
        presets: ["@babel/preset-env"]
        }
    }
}
```

### uglifyjs-webpack-plugin;打包时压缩js文件
#### npm i -D uglifyjs-webpack-plugin
```
const uglify = require('uglifyjs-webpack-plugin');
 plugins:[
        new uglify();
    ]
```

### 处理css文件的loader:css-loader,style-loader
#### npm install css-loader --save-dev 
#### npm install style-loader --save-dev
#### 功能是将css文件编译后打包插入到bundle.js文件中
```
//使用loader
module:{
    rules:[
        {
            test:/\.css$/,
            use:[
            {
                loader:'css-loader',
                options:{},
            },
            {
                loader:'style-loader',
                options:{},
            }
            ]
        }
    ]
}
```

### 文件处理loader：url-loader,file-loader
#### npm install --save-dev url-loader,npm install --save-dev file-loader
#### 处理一些外部引进的静态资源比如图片，比如css的引入图片时路径是相对css文件的，当转换为base64插入到html文件时路径是相对于html文件的，需要loader帮我们自动转化。并且转化成base64后插入到bundle.js减少网络请求,此时应该经将图片全部转化为base64不要做limit的限制
#### 使用url-loader时会有limit限制，这个选项就是限制图片的编译base64的限制。
#### 当limit限制以外的图片会按照outputPath这个选项打包进这个路径
```
//url-loader与file-loader功能相似
//此时外部文件资源变为了base64,打包时引入到了 bundle.js 中
module:{
    rules:[
           {
            //file-loader 解决css等文件中引入图片路径的问题
            //url-loader 当图片较小的时候会把图片BASE64编码，大于limit参数的时候还是使用file-loader 进行拷贝
            test: /\.(png|jpg|jpeg|gif|svg)$/,
            use: {
                loader: "url-loader",
                options: {
                    outputPath: "images/", // 图片输出的路径
                    limit: 1 * 1024
                }
            }
        }
    ]
}
```

### html页面标签引入图片怎么处理
#### 由于我们用loader编译css和js文件可以对其中的引入的外部资源的路径进行重新的编译，但是如果html页面使用<img>标签引用了外部资源打包时怎么办
#### 方法一：使用require引入路径
#### 方法二：使用html-loader
```
<img src = "${require(`./asset/timg.png`)}" alt="图片">
```

### CSS分离,extract-text-webpack-plugin
#### 在打包时我们希望css,less等不要打包到bundle.js文件中，而是单独的分离出来这是我们需要extract-text-webpack-plugin这个插件
#### 注意新版的webpack4.0 中这个插件支持的版本是npm install --save-dev extract-text-webpack-plugin@next
```
const path = require('path');
//引入glob
const glob = require('glob');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const ExtractTextPlugin = require("extract-text-webpack-plugin");
// 封装 entries 函数
var entries = function () {
    var jsDir = path.resolve(__dirname, 'src');
    //console.log(jsDir);
    var entryFiles = glob.sync(jsDir + '/*.{js,jsx}');
    //console.log(entryFiles);
    var map = {};
    for (var i = 0; i < entryFiles.length; i++) {
        var filePath = entryFiles[i];
        var filename = filePath.substring(filePath.lastIndexOf('\/') + 1, filePath.lastIndexOf('.'));
        map[filename] = filePath;
    }
    return map;
};
module.exports = {
    mode: "development",
    entry: entries(),
    output: {
        path: path.resolve(__dirname, 'dist'),
        publicPath: '/',
        filename: '[name].bundle.js',
    },
    devServer: {
        //告诉服务器从哪个目录中提供内容
        contentBase: path.resolve(__dirname, 'dist'),
        //服务器的IP地址，可以使用IP也可以使用localhost
        //当设置为0.0.0.0后将项目部署在服务器上启动测试的dev-server也可以进行访问
        host: 'localhost',
        //服务端压缩是否开启
        compress: true,
        //配置服务端口号
        port: 3000
    },
    module: {
        rules: [{
                test: /\.css$/,
                use: ExtractTextPlugin.extract({
                    fallback: "style-loader",
                    use: "css-loader"
                })
            }, {
                test: /\.(png|jpg|gif)$/,
                use: [{
                    loader: 'url-loader',
                    options: {
                        name: '../[name].[ext]',
                        limit: 1000000,
                    },
                }]
            },
            {
                test: /\.js$/,
                exclude: /(node_modules)/,
                use: {
                    loader: 'babel-loader',
                    options: {
                        presets: ['@babel/preset-env']
                    }
                }
            }
        ]
    },
    plugins: [
        new HtmlWebpackPlugin({
            minify: {
                removeAttributeQuotes: true
            },
            hash: true,
            template: './src/index.html'
        }),
        new ExtractTextPlugin("css/main.css"),
    ]
}
```

### less的使用与编译
#### 在使用less时需要安装less，编译less时需要使用less-loader
#### npm install --save-dev less-loader less
#### 编译后的less文件会被插入进css文件中
```
 {
    test: /\.less$/,
    use: ExtractTextPlugin.extract({
        use: [{
        loader: "css-loader"
        }, {
        loader: "less-loader"
        }],
        // use style-loader in development
        fallback: "style-loader"
    })
}
```

### 消除未使用的CSS：PurifyCSS-webpack插件
#### npm i -D purifycss-webpack purify-css
```
const PurifyCSSPlugin = require("purifycss-webpack");
new PurifyCSSPlugin({
      // Give paths to parse for rules. These should be absolute!
      paths: glob.sync(path.join(__dirname, 'src/*.html')),
})
```

### 引入第三方类库的方法
#### 局部的引入
```
//安装时需要注意的时Jquery最终要在生产环境中使用,npm install --save jquery;
//在入口文件main.js中引入 import $ from 'jquery';
//会将jquery打包进bundle.js出口文件中
//main.js
import main from './css/main.css';
import index from './css/index.less';
import img from './asset/timg.png';
import $ from 'jquery';
const app = document.getElementById('app');
app.innerText="Hello webpack!!!!"
let [a, b, c] = [1, 2, 3];
console.log(a);
console.log(b);
console.log(c);
$('#twobox').text('twobox使用了jquery');
```
#### 全局进入，在webpack.config.js中进入
#### 需要在webpack.config.js中进入webpack
#### 插件ProvidePlugin来使用第三类库
```
constc  webpack = require('webpack');
new webpack.ProvidePlugin({
    $:"jquery",
    jquery:"jquery"
})
```

### CleanWebpackPlugin 这个插件处理，打包时dist在每次构建之前清理文件夹，以便仅生成已使用的文件
#### npm install clean-webpack-plugin --save-dev
 ```
 //webpack.config.js
const CleanWebpackPlugin = require('clean-webpack-plugin');
 plugins: [
    new CleanWebpackPlugin(['dist'])
 ]
 ```
