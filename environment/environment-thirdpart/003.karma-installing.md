# Karma的安装

Jasmine是一个JS的测试工具，在Karma上运行Jasmine可完成JS的自动化测试、生成覆盖率报告等。本文不包含Jasmine的细节，主要介绍Karma的安装。

## 1. Karma介绍

Karma是Testacular的新名字，2012年Google开源了Testacular，2013年改名为Karma——它是一个让人感到很神秘的名字，表示佛教中的缘分，因果报应，比Cassandra这种名字更让人猜不透！它是一个基于Node.js的JavaScript测试执行过程管理工具（Test Runner），该工具可用于测试所有主流Web浏览器，也可集成到CI\(Continuous Integration）工具，也可和其他代码编辑器一起使用。这个测试工具的一个强大特征就是，它可以监控（Watch）文件的变化，然后自行执行，通过console.log显示测试结果。

Jasmine是单元测试框架，本文将介绍用Karma让Jasmine测试自动化完成，istanbul是一个单元测试代码覆盖率检查工具，可以很直观地告诉我们，单元测试对代码的控制程度。

## 2. Karma的安装

```bash
npm install -g karma
karma start
```

运行完成后就可以看到下边的输出：

```
19 04 2017 11:38:10.492:WARN [karma]: No captured browser, open http://localhost:9876/
19 04 2017 11:38:10.507:INFO [karma]: Karma v1.6.0 server started at http://0.0.0.0:9876/
```

## 3. Karma + Jasmine配置

最初需要初始化karma的配置文件：karma.conf.js，在项目中直接使用命令：`karma init`来初始化：

```bash
Which testing framework do you want to use ?
Press tab to list possible options. Enter to move to the next question.
> jasmine

Do you want to use Require.js ?
This will add Require.js plugin.
Press tab to list possible options. Enter to move to the next question.
> no

Do you want to capture any browsers automatically ?
Press tab to list possible options. Enter empty string to move to the next question.
> Chrome
>

What is the location of your source and test files ?
You can use glob patterns, eg. "js/*.js" or "test/**/*Spec.js".
Enter empty string to move to the next question.
> src/test/**/*Test.js
19 04 2017 11:41:58.262:WARN [init]: There is no file matching this pattern.

>

Should any of the files included by the previous patterns be excluded ?
You can use glob patterns, eg. "**/*.swp".
Enter empty string to move to the next question.
>

Do you want Karma to watch all the files and run the tests on change ?
Press tab to list possible options.
> yes


Config file generated at "/Users/lang/KTS/Source/Training/client/karma.conf.js".
```

安装集成包`karma-jasmine`

```
npm install karma-jasmine
```

## 4. 自动化单元测试

**3步准备工作**

1. 创建源文件：用于实现某种业务逻辑的文件，通常写得js脚本
2. 创建测试文件：符合jasmineAPI的测试js脚本
3. 修改karma.conf.js配置文件

**1.创建源文件**

用于实现某种业务逻辑的文件，就是通常写得JS脚本，如：

```javascript
~ vi src.js
function reverse(name){
    return name.split("").reverse().join("");
}
```

**2.创建测试文件**

```javascript
describe("A suite of basic functions", function (){
    it("reverse word",function(){
        expect("DCBA").toEqual(reverse("ABCD"));
    });
});
```

**3.修改karma.conf.js配置文件**

这里主要是修改files和exclude变量

```javascript
module.exports = function (config) {
    config.set({
        basePath: '',
        frameworks: ['jasmine'],
        files: ['*.js'],
        exclude: ['karma.conf.js'],
        reporters: ['progress'],
        port: 9876,
        colors: true,
        logLevel: config.LOG_INFO,
        autoWatch: true,
        browsers: ['Chrome'],
        captureTimeout: 60000,
        singleRun: false
    });
};
```

**启动karma**

单元测试全自动执行

```
karma start karma.conf.js
```

这样浏览器就会启动然后自动执行，如果修改了`test.js`由于配置文件中设置了autoWatch，所以test.js文件保存后，会自动执行单元测试。执行日志如下：提示我们单元测试出错了。

## 5. Karma和istanbul代码覆盖率

增加代码覆盖率检查和报告，增加istanbul依赖

```
npm install karma-coverage
```

修改karma.conf.js配置文件

```javascript
~ vi karma.conf.js

reporters: ['progress','coverage'],
preprocessors : {'src.js': 'coverage'},
coverageReporter: {
    type : 'html',
    dir : 'coverage/'
}
```

启动karma start，在工程目录下边找到`index.html`文件，coverage/chrome/index.html

用浏览器打开后，就可以看到代码覆盖率报告。

## 6. Karma第一次启动出现的问题

CHROME\_BIN的环境变量问题

```
~ D:\workspace\javascript\karma>karma start karma.conf.js
INFO [karma]: Karma v0.10.2 server started at http://localhost:9876/
INFO [launcher]: Starting browser Chrome
ERROR [launcher]: Cannot start Chrome
        Can not find the binary C:\Users\Administrator\AppData\Local\Google\Chrome\Application\chrome.exe
        Please set env variable CHROME_BIN
```

设置方法：找到系统中chrome的安装位置，找到chrome.exe文件：

```
~ D:\workspace\javascript\karma>set CHROME_BIN="C:\Program Files (x86)\Google\Chrome\Application\chrome.exe"
```



