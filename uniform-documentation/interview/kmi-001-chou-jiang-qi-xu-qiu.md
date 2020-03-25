# 抽奖器需求

使用前端技术实现一个简单的抽奖器，技术跟着公司本身的业务走，主要考虑使用以下内容，不同版本：

* React：[https://www.reactjs.org/](https://www.reactjs.org/) （可能需要翻墙，无法访问参考：[https://react.docschina.org/](https://react.docschina.org/) ）
* Ant Design：[https://ant.design/docs/react/introduce-cn](https://ant.design/docs/react/introduce-cn) 
* Zero UI：[http://www.vertxui.cn/](http://www.vertxui.cn/) （自研发开源框架，后边会使用，版本1和版本2中不考虑）

## 需求

使用React实现一个完整的抽奖器，基本需求：

* 设置40个数字，40个数字每0.2秒在界面切换一次。
* 右边设置一个按钮，每次点击按钮过后，数字停止，抽出四十个数字中随机的一个，并且将该数字显示在网页界面。
* 再次点击按钮过后，数字继续切换。

## 实现

* 版本一：使用React实现
* 版本二：使用Ant Design实现