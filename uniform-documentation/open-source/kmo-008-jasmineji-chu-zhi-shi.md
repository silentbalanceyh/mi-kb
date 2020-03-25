# Jasmine基础知识

TDD（Test Driven Development）测试驱动开发是敏捷开发中提出的最佳实践之一。jasmine很有意思地提出了BDD（Behavior Driven Development）行为驱动开发。测试驱动开发，对软件质量起了规范性的控制——未写实现，先写测试，一度成为Java领域中研发的圣经。随着JS兴起，功能越来越多，代码量越来越大，开发人员素质相差悬殊，真的有必要建立对代码的规范性控制。Jasmine就是为团队合作而生。

## 1. Jasmine介绍

Jasmine是一个用来编写JS测试的框架，它不依赖于任何其他的JS框架，也不需要对DOM，它拥有灵巧而明确的语法可以让您轻松编写测试代码。Jasmine本身结构很简单：

```javascript
describe("A suite", function() {
  var foo;
  beforeEach(function() {
    foo = 0;
    foo += 1;
  });

  afterEach(function() {
    foo = 0;
  });

  it("contains spec with an expectation", function() {
    expect(true).toBe(true);
  });
});
```

每个测试都在一个测试集中运行，Suite就是一个测试集，用describe函数封装，Spec表示每个测试用例，用it函数封装。通过expect函数，作为程序断言来判断相等关系。setup过程用beforeEach函数封装，tearDown过程用afterEach封装。

## 2. Jasmine安装

安装jasmine类库时，可直接使用bower一键搞定。

```
~ D:\workspace\javascript>mkdir jasmine
~ D:\workspace\javascript>cd jasmine

#查找jasmine项目
~ D:\workspace\javascript\jasmine>bower search jasmine
-----------------------------------------
Update available: 1.2.4 (current: 1.1.2)
Run npm update -g bower to update
-----------------------------------------

Search results:

    jasmine git://github.com/pivotal/jasmine.git
    jasmine-jquery git://github.com/velesin/jasmine-jquery
    jasmine-sinon git://github.com/froots/jasmine-sinon.git
    jasmine-ajax git://github.com/pivotal/jasmine-ajax.git
    jasmine.async git://github.com/derickbailey/jasmine.async.git
    flight-jasmine git://github.com/twitter/flight-jasmine.git
    jasmine-jstd-adapter git://github.com/ibolmo/jasmine-jstd-adapter.git
    jasmine-flight git://github.com/flightjs/jasmine-flight.git
    jasmine-signals git://github.com/AdamNowotny/jasmine-signals.git
    jasmine-underscore git://github.com/adscott/jasmine-underscore.git
    jasmine-bootstrap git://github.com/esbie/jasmine-bootstrap.git
    jasmine-data-provider git://github.com/sublimino/jasmine-data-provider.git
    jasmine-sugar git://github.com/fantactuka/jasmine-sugar.git

#安装jasmine类库
~ D:\workspace\javascript\jasmine>bower install jasmine
bower jasmine#*             not-cached git://github.com/pivotal/jasmine.git#*
bower jasmine#*                resolve git://github.com/pivotal/jasmine.git#*
bower jasmine#*               download https://github.com/pivotal/jasmine/archive/v1.3.1.tar.gz
bower jasmine#*                extract archive.tar.gz
bower jasmine#*               resolved git://github.com/pivotal/jasmine.git#1.3.1
bower jasmine#~1.3.1           install jasmine#1.3.1

jasmine#1.3.1 bower_components\jasmine
```

## 3. Jasmine环境配置

jasmine运行需要四个部分：

* 运行时环境：这里基于Chrome浏览器，通过HTML作为JS载体
* 源文件：用于实现某种逻辑的文件，就是平时写得JS脚本
* 测试文件：符合JasmineAPI的测试JS脚本
* 输出结果：jasmine提供了基于网页的输出结果

使用流程：

1. 新建一个HTML文件：`test.html`

```html
~ vi test.html

<!DOCTYPE html>
<html>
<head>
<title>jasmine test</title>
<link rel="stylesheet" type="text/css" href="bower_components/jasmine/lib/jasmine-core/jasmine.css">
<script type="text/javascript" src="bower_components/jasmine/lib/jasmine-core/jasmine.js"></script>
<script type="text/javascript" src="bower_components/jasmine/lib/jasmine-core/jasmine-html.js"></script>
</head>
<body>
<h1>jasmine test</h1>
<script type="text/javascript" src="src.js"></script>
<script type="text/javascript" src="test.js"></script>
<script type="text/javascript" src="report.js"></script>
</body>
</html>
```

我们看到页面上有5个JS导入：

* `jasmine.js`：核心用于执行单元测试的类库【直接导入】
* `jasmine-html.js`：用于显示单元测试的结果类库【直接导入】
* `src.js`：我们自己的业务逻辑JS
* `test.js`：单元测试的JS
* `report.js`：用于启动单元测试JS【启动脚本，写法固定】

```javascript
~ vi report.js

(function() {
    var jasmineEnv = jasmine.getEnv();
    jasmineEnv.updateInterval = 1000;

    var htmlReporter = new jasmine.HtmlReporter();

    jasmineEnv.addReporter(htmlReporter);

    jasmineEnv.specFilter = function(spec) {
        return htmlReporter.specFilter(spec);
    };

    var currentWindowOnload = window.onload;

    window.onload = function() {
        if (currentWindowOnload) {
            currentWindowOnload();
        }
        execJasmine();
    };

    function execJasmine() {
        jasmineEnv.execute();
    }

})();
```

`src.js`是实现业务逻辑的文件，我们定义sayHello的函数

```javascript
~ vi src.js

function sayHello(name){
    return "Hello " + name;
}
```

`test.js`则是针对源文件的单元测试

```javascript
~ vi test.js

describe("A suite of basic functions", function() {
    var name;

    it("sayHello", function() {
        name = "Conan";
        var exp = "Hello Conan";
        expect(exp).toEqual(sayHello(name));
    });
});
```

## 4. Jasmine使用

**1）测试先行：**

测试先行，就是没写实现，先写用例。刚刚已经配置好了jasmine的环境，后边将所有功能代码都写在`src.js`中，测试代码都写在`test.js`中。有一个需求，要实现单词翻转功能。

编辑`test.js`

```javascript
~ vi test.js

it("reverse word",function(){
    expect("DCBA").toEqual(reverse("ABCD"));
});
```

再执行该页面，测试会失败，因为没有reverse函数……

**2）Jasmine语法实践**

以下内容是对Jasmine语法的介绍，都在`test.js`中实现，做一个嵌套的`describe("A suite of jasmine's function")`

对断言表达式的使用

```javascript
describe("A suite of jasmine's function", function() {
    describe("Expectations",function(){
        it("Expectations",function(){
            expect("AAA").toEqual("AAA");
            expect(52.78).toMatch(/\d*.\d\d/);
            expect(null).toBeNull();
            expect("ABCD").toContain("B");
            expect(52,78).toBeLessThan(99);
            expect(52.78).toBeGreaterThan(18);

            var x = true;
            var y;
            expect(x).toBe(true);
            expect(x).toBeDefined();
            expect(y).toBeUndefined();
            expect(x).toBeTruthy();
            expect(!x).toBeFalsy();

            var fun = function() { return a + 1;};
            expect(fun).toThrow();
        });
    });
});
```

对开始前和使用后的变量赋值

```javascript
    describe("Setup and Teardown",function(){
        var foo;
        beforeEach(function() {
            foo = 0;
            foo += 1;
        });
        afterEach(function() {
            foo = 0;
        });

        it("is just a function, so it can contain any code", function() {
            expect(foo).toEqual(1);
        });

        it("can have more than one expectation", function() {
            expect(foo).toEqual(1);
            expect(true).toEqual(true);
        });
    });
```

对忽略suite的声明

```javascript
    describe("Disabling Specs and Suites", function() {
        it("Disabling Specs and Suites",function(){
            expect("AAA").toEqual("AAA");
        });
    });
```

对异步程序的单元测试

```javascript
    describe("Asynchronous Support",function(){
        var value, flag;
        it("Asynchronous Support", function() {
            runs(function() {
                flag = false;
                value = 0;
                setTimeout(function() {
                    flag = true;
                }, 500);
            });
            waitsFor(function() {
                value++;
                return flag;
            }, "The Value should be incremented", 750);

            runs(function() {
                expect(value).toBeGreaterThan(0);
            });
        });
    });
```

这样就成功对JS完成了对应单元测试。

