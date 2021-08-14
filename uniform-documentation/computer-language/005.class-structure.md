## Class字节码结构

### __代码清单__
包：`com.sco.core`<br/>
类清单：<br/>

* `com.sco.core.TestClass`

### __javac编译过后的字节码（16进制）__

下边的截图就是.class文件的内容<br/>
![com.sco.core.TestClass编译后结果](/assets/images/kml/005/1-1.JPG)<br/>
它对应的源代码部分的内容为<br/>
![com.sco.core.TestClass源代码](/assets/images/kml/005/1-2.JPG)<br/>

<hr/>

### __核心概念__

Java虚拟机规范中规定，Class文件格式采用一种类似C语言结构体的伪结构来存储，它只有两种数据类型

* *无符号数（基本数据类型）*<br/>主要用于描述数字、索引引用、数量值、或UTF-8编码构成的字符串；<br/>
u1 -- 1个字节<br/>u2 -- 2个字节<br/>u4 -- 4个字节<br/>u8 -- 8个字节
* *表（符合数据类型）*<br/>用于描述有层次关系的符合结构的数据；<br/>
习惯性以“`_info`”结尾

#### __1.Class文件格式__

<table style="border-collapse:collapse;font-size:12px;">
	<tr style="font-style:italic">
		<td style="width:80px;">数据类型</td>
		<td>名称</td>
		<td>数量</td>
	</tr>
	<tr>
		<td><font style="color:red;">u4</font></td>
		<td>magic</td>
		<td>1</td>
	</tr>
	<tr>
		<td><font style="color:red;">u2</font></td>
		<td>minor_version</td>
		<td>1</td>
	</tr>
	<tr>
		<td><font style="color:red;">u2</font></td>
		<td>major_version</td>
		<td>1</td>
	</tr>
	<tr>
		<td><font style="color:red;">u2</font></td>
		<td>constant_pool_count</td>
		<td>1</td>
	</tr>
	<tr>
		<td><font style="color:red;">cp_info</font></td>
		<td>constant_pool</td>
		<td>constant_pool_count + 1</td>
	</tr>
	<tr>
		<td><font style="color:red;">u2</font></td>
		<td>access_flags</td>
		<td>1</td>
	</tr>
	<tr>
		<td><font style="color:red;">u2</font></td>
		<td>this_class</td>
		<td>1</td>
	</tr>
	<tr>
		<td><font style="color:red;">u2</font></td>
		<td>super_class</td>
		<td>1</td>
	</tr>
	<tr>
		<td><font style="color:red;">u2</font></td>
		<td>interfaces_count</td>
		<td>1</td>
	</tr>
	<tr>
		<td><font style="color:red;">u2</font></td>
		<td>interfaces</td>
		<td>interfaces_count</td>
	</tr>
	<tr>
		<td><font style="color:red;">u2</font></td>
		<td>fields_count</td>
		<td>1</td>
	</tr>
	<tr>
		<td><font style="color:red;">field_info</font></td>
		<td>fields</td>
		<td>fields_count</td>
	</tr>
	<tr>
		<td><font style="color:red;">u2</font></td>
		<td>methods_count</td>
		<td>1</td>
	</tr>
	<tr>
		<td><font style="color:red;">method_info</font></td>
		<td>methods</td>
		<td>methods_count</td>
	</tr>
	<tr>
		<td><font style="color:red;">u2</font></td>
		<td>attributes_count</td>
		<td>1</td>
	</tr>
	<tr>
		<td><font style="color:red;">attribute_info</font></td>
		<td>attributes</td>
		<td>attributes_count</td>
	</tr>
</table>

#### __2.Class文件版本号__

##### __2.1.Java中`javac`的参数`-source`和`-target`__

* -source<br/>
表示当前源代码使用什么版本下的JDK进行编译，例如：<font style="color:white;background-color:black">javac -source 1.4 TestClass.java</font>表示使用JDK 1.4的语法对当前.java源文件进行编译（我机器安装的JDK为1.8），估计从JDK 1.9开始就不支持1.5/5以及更早版本了；
* -target<br/>
表示编译器生成特定版本的Java类文件格式，可指定Class文件格式，例如：<font style="color:white;background-color:black">javac -source 1.4 -target 1.4 TestClass.java</font>表示使用-source 1.4的语法源代码，生成的最终Class文件格式也是1.4的格式；

*-target在使用的时候需要加上-source*，否则会产生错误，下边两种错误都是不能正确生成.class字节码文件的：<br/>
<font style="color:red">错误1</font>：（不带-source）默认的-source是1.8，但在输出类文件格式的时候尝试使用1.5的文件格式输出<br/>
![不带-source](/assets/images/kml/005/1-3.JPG)<br/>
<font style="color:red">错误2</font>：（带-source）带上了-source的1.8参数过后尝试用1.5的文件格式输出<br/>
![带-source](/assets/images/kml/005/1-4.JPG)<br/>
<font style="color:blue">警告</font>：（带-source）带上了-source的1.5参数过后尝试用1.7的文件格式输出（编译可通过）<br/>
![警告](/assets/images/kml/005/1-5.JPG)<br/>

*综上所述，-source的版本号必须小于或等于-target的版本号，一旦大于了过后可能导致编译不通过，但这里会有一个问题，从下边的表看来，直接使用低版本输出-target <version>的方式应该是可行的，但这个用法似乎只适合特定版本的JDK，例如：1.6.0_01可直接使用-target 1.5输出JDK 1.5的字节码文件，我在本机使用1.8的版本输出时就会报错。（ -source <= -target ）*

##### __2.2.Class文件版本号__

<table style="border-collapse:collapse;font-size:12px;">
	<tr style="font-style:italic">
		<td style="width:80px;">编译器版本</td>
		<td>-target参数</td>
		<td>十六进制版本</td>
		<td>十进制版本</td>
	</tr>
	<tr>
		<td>JDK 1.1.8</td>
		<td>不能带target参数</td>
		<td>00 03 00 2D</td>
		<td><font style="color:red;">45.3</font></td>
	</tr>
	<tr>
		<td>JDK 1.2.2</td>
		<td>不带（默认 -target 1.1）</td>
		<td>00 03 00 2D</td>
		<td><font style="color:red;">45.3</font></td>
	</tr>
	<tr>
		<td>JDK 1.2.2</td>
		<td>-target 1.2</td>
		<td>00 00 00 2E</td>
		<td><font style="color:red;">46.0</font></td>
	</tr>
	<tr>
		<td>JDK 1.3.1_19</td>
		<td>不带（默认 -target 1.1）</td>
		<td>00 00 00 2D</td>
		<td><font style="color:red;">45.3</font></td>
	</tr>
	<tr>
		<td>JDK 1.3.1_19</td>
		<td>-target 1.3</td>
		<td>00 00 00 2F</td>
		<td><font style="color:red;">47.0</font></td>
	</tr>
	<tr>
		<td>JDK 1.4.2_10</td>
		<td>不带（默认 -target 1.2）</td>
		<td>00 00 00 2E</td>
		<td><font style="color:red;">46.0</font></td>
	</tr>
	<tr>
		<td>JDK 1.4.2_10</td>
		<td>-target 1.4</td>
		<td>00 00 00 30</td>
		<td><font style="color:red;">48.0</font></td>
	</tr>
	<tr>
		<td>JDK 1.5.0_11</td>
		<td>不带（默认 -target 1.5）</td>
		<td>00 00 00 31</td>
		<td><font style="color:red;">49.0</font></td>
	</tr>
	<tr>
		<td>JDK 1.5.0_11</td>
		<td>-target 1.4 -source 1.4</td>
		<td>00 00 00 30</td>
		<td><font style="color:red;">48.0</font></td>
	</tr>
	<tr>
		<td>JDK 1.6.0_01</td>
		<td>不带（默认 -target 1.6）</td>
		<td>00 00 00 32</td>
		<td><font style="color:red;">50.0</font></td>
	</tr>
	<tr>
		<td>JDK 1.6.0_01</td>
		<td>-target 1.5</td>
		<td>00 00 00 31</td>
		<td><font style="color:red;">49.0</font></td>
	</tr>
	<tr>
		<td>JDK 1.6.0_01</td>
		<td>-target 1.4 -source 1.4</td>
		<td>00 00 00 30</td>
		<td><font style="color:red;">48.0</font></td>
	</tr>
	<tr>
		<td>JDK 1.7.0</td>
		<td>不带（默认 -target 1.7）</td>
		<td>00 00 00 33</td>
		<td><font style="color:red;">51.0</font></td>
	</tr>
	<tr>
		<td>JDK 1.7.0</td>
		<td>-target 1.6</td>
		<td>00 00 00 32</td>
		<td><font style="color:red;">50.0</font></td>
	</tr>
	<tr>
		<td>JDK 1.7.0</td>
		<td>-target 1.4 -source 1.4</td>
		<td>00 00 00 30</td>
		<td><font style="color:red;">48.0</font></td>
	</tr>
	<tr>
		<td>JDK 1.8.0</td>
		<td>不带（默认 -target 1.8）</td>
		<td>00 00 00 34</td>
		<td><font style="color:red;">52.0</font></td>
	</tr>
</table>

注意：

* -target 1.1中包含了次版本号，之后就没有次版本号了；
* 从1.1到1.4的语法差异很小，默认的-target使用的都不是自身对应版本；
* 1.5开始过后默认的-target是1.5，所以如果要生成1.4的文件格式则需要加上-source 1.4，之后的JDK使用也如此；

最后：<font style="color:red">某个JVM能接受的class文件的最大主版本号不能超过对应JDK带相应-target参数编译出来的class文件的版本号。</font> 例：1.4的JVM能接受最大的class文件的主版本号不能超过1.4 JDK使用-target 1.4输出的主版本号，即48。因为JDK 1.5默认编译输出-target为1.5，则最终class字节码是49.0，所以1.4的JVM是无法执行和支持JDK 1.5编译输出的字节码的，只有抛出错误。

#### __3.关于常量池__

##### __3.1.基础知识__
常量池中主要存放两大类型常量。

* __字面量【Literal】__<br/>
* __符号引用【Symbolic References】__（详细内容可参考编译原理）<br/>
符号引用主要包含三种：<br/>1）类和接口的全限定名（Fully Qualified Name）；<br/>2）字段的名称和描述符（Descriptor）；<br/>3）方法的名称和描述符；

Java和C/C++语言有一点不同，它没有Link【链接】的步骤。

* C/C++语言一般是把源文件编译成.obj的目标文件，然后“链接”成可执行程序；
* Java则会使用JVM加载.class文件，在加载的时候使用动态链接，也就是说Class文件不会保存方法和字段的最终内存信息，这些符号引用如果不经过转化的话是无法直接被虚拟机使用的。

##### __3.2.常量项目类型__
常量池中每一项常量都是一个表，共有11种结构【除去JDK 1.7之后的CONSTANT\_InvokeDynamic和CONSTANT\_InvokeDynamicTrans两个】，这11种表的第一位都是一个u1类型的标志位（Tag，1 ~ 12，缺少标志为2的数据类型），表示当前常量的类型。

<table style="border-collapse:collapse;font-size:12px;">
	<tr style="font-style:italic">
		<td style="width:240px;">类型</td>
		<td>标志</td>
		<td>描述</td>
	</tr>
	<tr>
		<td><font style="color:red;">CONSTANT_Utf8_info</font></td>
		<td>1</td>
		<td>UTF-8编码的字符串</td>
	</tr>
	<tr>
		<td><font style="color:red;">CONSTANT_Integer_info</font></td>
		<td>3</td>
		<td>整型字面量</td>
	</tr>
	<tr>
		<td><font style="color:red;">CONSTANT_Float_info</font></td>
		<td>4</td>
		<td>浮点型字面量</td>
	</tr>
	<tr>
		<td><font style="color:red;">CONSTANT_Long_info</font></td>
		<td>5</td>
		<td>长整型字面量</td>
	</tr>
	<tr>
		<td><font style="color:red;">CONSTANT_Double_info</font></td>
		<td>6</td>
		<td>双精度浮点型字面量</td>
	</tr>
	<tr>
		<td><font style="color:red;">CONSTANT_Class_info</font></td>
		<td>7</td>
		<td>类或接口的符号引用</td>
	</tr>
	<tr>
		<td><font style="color:red;">CONSTANT_String_info</font></td>
		<td>8</td>
		<td>字符串类型字面量</td>
	</tr>
	<tr>
		<td><font style="color:red;">CONSTANT_Fieldref_info</font></td>
		<td>9</td>
		<td>字段的符号引用</td>
	</tr>
	<tr>
		<td><font style="color:red;">CONSTANT_Methodref_info</font></td>
		<td>10</td>
		<td>类中方法的符号引用</td>
	</tr>
	<tr>
		<td><font style="color:red;">CONSTANT_InterfaceMethodref_info</font></td>
		<td>11</td>
		<td>接口中方法的符号引用</td>
	</tr>
	<tr>
		<td><font style="color:red;">CONSTANT_NameAndType_info</font></td>
		<td>12</td>
		<td>字段或方法的部分符号引用</td>
	</tr>
</table>

##### __3.3.使用javap输出常量__

JDK中提供了javap工具，该工具主要用于分析字节码，使用下边命令可输出当前字节码文件中的所有常量（例子中<font style="color:red">35项</font>）：<br/>
<font style="color:white;background-color:black">javap -verbose TestClass.class</font><br/>
输出（为了方便截图使用PowerShell输出，路径切换成.\TestClass.class，其他的没有变化）：
![常量表](/assets/images/kml/005/1-6.JPG)<br/>
![常量表](/assets/images/kml/005/1-7.JPG)<br/>

##### __3.4.常量类型的结构总表__

上边提到了11种常量池的结构信息，那么这里再提供11种常量类型的结构总表，细化到前边提到的数据类型（Tag对应3.2中的表）。

<table style="border-collapse:collapse;font-size:12px;">
	<tr style="font-style:italic">
		<td style="width:240px;">常量</td>
		<td>项目</td>
		<td>类型</td>
		<td>描述</td>
	</tr>
	<tr>
		<td rowspan=3><font style="color:red;">CONSTANT_Utf8_info</font></td>
		<td>tag</td>
		<td><font style="color:blue;">u1</font></td>
		<td>值为1</td>
	</tr>
	<tr>
		<td>length</td>
		<td><font style="color:blue;">u2</font></td>
		<td>UTF-8编码的字符串占用的字节数</td>
	</tr>
	<tr>
		<td>bytes</td>
		<td><font style="color:blue;">u1</font></td>
		<td>长度为length的UTF-8编码的字符串</td>
	</tr>
	<tr>
		<td rowspan=2><font style="color:red;">CONSTANT_Integer_info</font></td>
		<td>tag</td>
		<td><font style="color:blue;">u1</font></td>
		<td>值为3</td>
	</tr>
	<tr>
		<td>bytes</td>
		<td><font style="color:blue;">u4</font></td>
		<td>按照高位在前存储的int值</td>
	</tr>
	<tr>
		<td rowspan=2><font style="color:red;">CONSTANT_Float_info</font></td>
		<td>tag</td>
		<td><font style="color:blue;">u1</font></td>
		<td>值为4</td>
	</tr>
	<tr>
		<td>bytes</td>
		<td><font style="color:blue;">u4</font></td>
		<td>按照高位在前存储的float值</td>
	</tr>
	<tr>
		<td rowspan=2><font style="color:red;">CONSTANT_Long_info</font></td>
		<td>tag</td>
		<td><font style="color:blue;">u1</font></td>
		<td>值为5</td>
	</tr>
	<tr>
		<td>bytes</td>
		<td><font style="color:blue;">u8</font></td>
		<td>按照高位在前存储的long值</td>
	</tr>
	<tr>
		<td rowspan=2><font style="color:red;">CONSTANT_Double_info</font></td>
		<td>tag</td>
		<td><font style="color:blue;">u1</font></td>
		<td>值为6</td>
	</tr>
	<tr>
		<td>bytes</td>
		<td><font style="color:blue;">u8</font></td>
		<td>按照高位在前存储的double值</td>
	</tr>
	<tr>
		<td rowspan=2><font style="color:red;">CONSTANT_Class_info</font></td>
		<td>tag</td>
		<td><font style="color:blue;">u1</font></td>
		<td>值为7</td>
	</tr>
	<tr>
		<td>index</td>
		<td><font style="color:blue;">u2</font></td>
		<td>指向全限定名常量项的索引</td>
	</tr>
	<tr>
		<td rowspan=2><font style="color:red;">CONSTANT_String_info</font></td>
		<td>tag</td>
		<td><font style="color:blue;">u1</font></td>
		<td>值为8</td>
	</tr>
	<tr>
		<td>index</td>
		<td><font style="color:blue;">u2</font></td>
		<td>指向字符串字面量的索引</td>
	</tr>
	<tr>
		<td rowspan=3><font style="color:red;">CONSTANT_Fieldref_info</font></td>
		<td>tag</td>
		<td><font style="color:blue;">u1</font></td>
		<td>值为9</td>
	</tr>
	<tr>
		<td>index</td>
		<td><font style="color:blue;">u2</font></td>
		<td>指向声明字段的类或接口描述符CONSTANT_Class_info的索引项</td>
	</tr>
	<tr>
		<td>index</td>
		<td><font style="color:blue;">u2</font></td>
		<td>指向字段描述符CONSTANT_NameAndType的索引项</td>
	</tr>
	<tr>
		<td rowspan=3><font style="color:red;">CONSTANT_Methodref_info</font></td>
		<td>tag</td>
		<td><font style="color:blue;">u1</font></td>
		<td>值为10</td>
	</tr>
	<tr>
		<td>index</td>
		<td><font style="color:blue;">u2</font></td>
		<td>指向声明方法的类或接口描述符CONSTANT_Class_info的索引项</td>
	</tr>
	<tr>
		<td>index</td>
		<td><font style="color:blue;">u2</font></td>
		<td>指向名称及类型CONSTANT_NameAndType的索引项</td>
	</tr>
	<tr>
		<td rowspan=3><font style="color:red;">CONSTANT_InterfaceMethodref_info</font></td>
		<td>tag</td>
		<td><font style="color:blue;">u1</font></td>
		<td>值为11</td>
	</tr>
	<tr>
		<td>index</td>
		<td><font style="color:blue;">u2</font></td>
		<td>指向声明字段的类或接口描述符CONSTANT_Class_info的索引项</td>
	</tr>
	<tr>
		<td>index</td>
		<td><font style="color:blue;">u2</font></td>
		<td>指向名称及类型CONSTANT_NameAndType的索引项</td>
	</tr>
	<tr>
		<td rowspan=3><font style="color:red;">CONSTANT_NameAndType_info</font></td>
		<td>tag</td>
		<td><font style="color:blue;">u1</font></td>
		<td>值为12</td>
	</tr>
	<tr>
		<td>index</td>
		<td><font style="color:blue;">u2</font></td>
		<td>指向该字段或方法名称常量项的索引</td>
	</tr>
	<tr>
		<td>index</td>
		<td><font style="color:blue;">u2</font></td>
		<td>指向该字段或方法描述符常量项的索引</td>
	</tr>
</table>

#### __4.访问标志__
当常量池结束过后，接着的2个字节就表示访问标记（access\_flags），这个标记用于标识类或者接口层次的访问信息，例如：

1. 这个Class是类还是接口？
2. 是否定义为public类型？
3. 是否定义为abstract类型？
4. 如果是类，有没有被声明为final？

##### __4.1.访问标志表__

<table style="border-collapse:collapse;font-size:12px;">
	<tr style="font-style:italic">
		<td style="width:120px;">标志名称</td>
		<td>标志值</td>
		<td>二进制值</td>
		<td>含义</td>
	</tr>
	<tr>
		<td><font style="color:red;">ACC_PUBLIC</font></td>
		<td><font style="color:blue;">0x0001</font></td>
		<td>0000 0000 0000 0001</td>
		<td>是否为public类型</td>
	</tr>
	<tr>
		<td><font style="color:red;">ACC_FINAL</font></td>
		<td><font style="color:blue;">0x0010</font></td>
		<td>0000 0000 0001 0000</td>
		<td>是否被声明为final，只有类可设置</td>
	</tr>
	<tr>
		<td><font style="color:red;">ACC_SUPER</font></td>
		<td><font style="color:blue;">0x0020</font></td>
		<td>0000 0000 0010 0000</td>
		<td>是否允许使用invokespecial字节码指令，JDK 1.2之后编译出来的类的这个标志为真</td>
	</tr>
	<tr>
		<td><font style="color:red;">ACC_INTERFACE</font></td>
		<td><font style="color:blue;">0x0200</font></td>
		<td>0000 0010 0000 0000</td>
		<td>标识这个是一个接口</td>
	</tr>
	<tr>
		<td><font style="color:red;">ACC_ABSTRACT</font></td>
		<td><font style="color:blue;">0x0400</font></td>
		<td>0000 0100 0000 0000</td>
		<td>是否为abstract类型，对于接口或抽象类来说，此标记值为真，其他类值为假</td>
	</tr>
	<tr>
		<td><font style="color:red;">ACC_SYNTHETIC</font></td>
		<td><font style="color:blue;">0x1000</font></td>
		<td>0001 0000 0000 0000</td>
		<td>（JDK 1.5之后定义）标识这个类并非由用户代码生成</td>
	</tr>
	<tr>
		<td><font style="color:red;">ACC_ANNOTATION</font></td>
		<td><font style="color:blue;">0x2000</font></td>
		<td>0010 0000 0000 0000</td>
		<td>（JDK 1.5之后定义）标识这是一个注解</td>
	</tr>
	<tr>
		<td><font style="color:red;">ACC_ENUM</font></td>
		<td><font style="color:blue;">0x4000</font></td>
		<td>0100 0000 0000 0000</td>
		<td>（JDK 1.5之后定义）标识这是一个枚举</td>
	</tr>
</table>

##### __4.2.注意事项和访问标志计算__
<font style="color:red">*其他标记比较容易理解，这里解释一下ACC_SYNTHETIC标记，ACC\_SYNTHETIC标记等价的属性称为Synthetic Attribute，它用于指示当前类、接口、方法或字段由编译器生成，而不在源代码中存在（不包含类初始化函数和实例初始化函数），相同的功能还有一种方式就是在类、接口、方法或字段的访问权限中设置ACC\_SYNTHETIC标记。Synthetic Attribute是从JDK 1.1中引入的，主要用于支持内嵌类和接口（Nested classes && Interfaces），这些功能目前都可以使用ACC\_SYNTHETIC来表达。ACC\_SYNTHETIChe Synthetic Attribute功能相同，但不是同一个东西。*</font>

access_flags的计算公式为：`access_flags = flagA | flagB | flagB ...`<br/>
比如：一个访问访问标志是`0x0021 = 0x0020 | 0x0001 = ACC_SUPER | ACC_PUBLIC`

#### __5.字段表集合__
字段表（field\_info）用于描述类或接口中声明的变量，它包含类变量、实例变量，但不包括方法内的局部变量和块变量。和cp\_info部分不一样，cp\_info因为常量类型的不一样其数据结构有11种，但field\_info的结构只有一种。

##### __5.1.字段表结构__

<table style="border-collapse:collapse;font-size:12px;">
	<tr style="font-style:italic">
		<td style="width:100px;">类型</td>
		<td>名称</td>
		<td>数量</td>
	</tr>
	<tr>
		<td><font style="color:blue;">u2</font></td>
		<td><font style="color:red;">access_flags</font></td>
		<td>1</td>
	</tr>
	<tr>
		<td><font style="color:blue;">u2</font></td>
		<td><font style="color:red;">name_index</font></td>
		<td>1</td>
	</tr>
	<tr>
		<td><font style="color:blue;">u2</font></td>
		<td><font style="color:red;">descriptor_index</font></td>
		<td>1</td>
	</tr>
	<tr>
		<td><font style="color:blue;">u2</font></td>
		<td><font style="color:red;">attributes_count</font></td>
		<td>1</td>
	</tr>
	<tr>
		<td><font style="color:blue;">attribute_info</font></td>
		<td><font style="color:red;">attributes</font></td>
		<td>attributes_count</td>
	</tr>
</table>

##### __5.2.字段访问标志__
字段访问标志和类的访问标志算法是一样，但因为修饰字段的标志和修饰类的标志不太一样，看看下边的字段访问标志（上表结构中的access\_flags）。

<table style="border-collapse:collapse;font-size:12px;">
	<tr style="font-style:italic">
		<td style="width:120px;">标志名称</td>
		<td>标志值</td>
		<td>二进制值</td>
		<td>含义</td>
	</tr>
	<tr>
		<td><font style="color:red;">ACC_PUBLIC</font></td>
		<td><font style="color:blue;">0x0001</font></td>
		<td>0000 0000 0000 0001</td>
		<td>是否public</td>
	</tr>
	<tr>
		<td><font style="color:red;">ACC_PRIVATE</font></td>
		<td><font style="color:blue;">0x0002</font></td>
		<td>0000 0000 0000 0010</td>
		<td>是否private</td>
	</tr>
	<tr>
		<td><font style="color:red;">ACC_PROTECTED</font></td>
		<td><font style="color:blue;">0x0004</font></td>
		<td>0000 0000 0000 0100</td>
		<td>是否protected</td>
	</tr>
	<tr>
		<td><font style="color:red;">ACC_STATIC</font></td>
		<td><font style="color:blue;">0x0008</font></td>
		<td>0000 0000 0000 1000</td>
		<td>是否static</td>
	</tr>
	<tr>
		<td><font style="color:red;">ACC_FINAL</font></td>
		<td><font style="color:blue;">0x0010</font></td>
		<td>0000 0000 0001 0000</td>
		<td>是否final</td>
	</tr>
	<tr>
		<td><font style="color:red;">ACC_VOLATILE</font></td>
		<td><font style="color:blue;">0x0040</font></td>
		<td>0000 0000 0100 0000</td>
		<td>是否volatile</td>
	</tr>
	<tr>
		<td><font style="color:red;">ACC_TRANSIENT</font></td>
		<td><font style="color:blue;">0x0080</font></td>
		<td>0000 0000 1000 0000</td>
		<td>是否transient</td>
	</tr>
	<tr>
		<td><font style="color:red;">ACC_SYNTHETIC</font></td>
		<td><font style="color:blue;">0x1000</font></td>
		<td>0001 0000 0000 0000</td>
		<td>是否由编译器自动产生</td>
	</tr>
	<tr>
		<td><font style="color:red;">ACC_ENUM</font></td>
		<td><font style="color:blue;">0x4000</font></td>
		<td>0100 0000 0000 0000</td>
		<td>是否enum</td>
	</tr>
</table>

##### __5.3.简单名称、描述符、全限定名__
在access\_flags标志之后的有两部分：

* name\_index：表示字段的简单名称；
* descriptor\_index：表示字段和方法的描述符；

这里区分三个概念，本文中反复提到：全限定名、简单名称、描述符：

1. __全限定名__<br/>
全限定名格式如：`com/sco/core/TestClass`，仅仅是把类全名中的`.`替换成了`/`而已，为了连续多个全限定名不混淆，结尾会使用一个`;`表示全限定名结束。
2. __简单名称__<br/>
简单名称则是没有类型、参数修饰的方法或字段名，比如TestClass类中的`age`，`name`字段名，`inc`方法名。
3. __描述符__<br/>
方法和字段的描述符主要用来描述字段的数据类型、方法的参数列表（包括数量、类型、顺序）和返回值。

##### __5.4.字段描述符表__

<table style="border-collapse:collapse;font-size:12px;">
	<tr style="font-style:italic">
		<td style="width:100px;">标识字符</td>
		<td>十六进制值</td>
		<td>含义</td>
	</tr>
	<tr>
		<td><font style="color:blue;">B</font></td>
		<td><font style="color:red;">42</font></td>
		<td>基本类型byte</td>
	</tr>
	<tr>
		<td><font style="color:blue;">C</font></td>
		<td><font style="color:red;">43</font></td>
		<td>基本类型char</td>
	</tr>
	<tr>
		<td><font style="color:blue;">D</font></td>
		<td><font style="color:red;">44</font></td>
		<td>基本类型double</td>
	</tr>
	<tr>
		<td><font style="color:blue;">F</font></td>
		<td><font style="color:red;">46</font></td>
		<td>基本类型float</td>
	</tr>
	<tr>
		<td><font style="color:blue;">I</font></td>
		<td><font style="color:red;">49</font></td>
		<td>基本类型int</td>
	</tr>
	<tr>
		<td><font style="color:blue;">J</font></td>
		<td><font style="color:red;">4A</font></td>
		<td>基本类型long</td>
	</tr>
	<tr>
		<td><font style="color:blue;">S</font></td>
		<td><font style="color:red;">53</font></td>
		<td>基本类型short</td>
	</tr>
	<tr>
		<td><font style="color:blue;">Z</font></td>
		<td><font style="color:red;">5A</font></td>
		<td>基本类型boolean</td>
	</tr>
	<tr>
		<td><font style="color:blue;">V</font></td>
		<td><font style="color:red;">56</font></td>
		<td>基本类型void</td>
	</tr>
	<tr>
		<td><font style="color:blue;">L</font></td>
		<td><font style="color:red;">4C</font></td>
		<td>对象类型，如：Ljava/lang/Object;</td>
	</tr>
</table>
除了上述的基本类型和对象类型描述符以外，Java中还有其他数据类型的描述符：

__*数组类型*__：对于数组类型，每一维度使用一个前置的“`[`”字符来描述，例：java.lang.String\[\]\[\] => [[Ljava/lang/String; int[] => [I；

#### __6.方法表集合__
方法表集合（method\_info）和字段表集合的结构是一致的，只是访问标志不同。

##### __6.1.方法访问标志__

<table style="border-collapse:collapse;font-size:12px;">
	<tr style="font-style:italic">
		<td style="width:120px;">标志名称</td>
		<td>标志值</td>
		<td>二进制值</td>
		<td>含义</td>
	</tr>
	<tr>
		<td><font style="color:red;">ACC_PUBLIC</font></td>
		<td><font style="color:blue;">0x0001</font></td>
		<td>0000 0000 0000 0001</td>
		<td>是否public</td>
	</tr>
	<tr>
		<td><font style="color:red;">ACC_PRIVATE</font></td>
		<td><font style="color:blue;">0x0002</font></td>
		<td>0000 0000 0000 0010</td>
		<td>是否private</td>
	</tr>
	<tr>
		<td><font style="color:red;">ACC_PROTECTED</font></td>
		<td><font style="color:blue;">0x0004</font></td>
		<td>0000 0000 0000 0100</td>
		<td>是否protected</td>
	</tr>
	<tr>
		<td><font style="color:red;">ACC_STATIC</font></td>
		<td><font style="color:blue;">0x0008</font></td>
		<td>0000 0000 0000 1000</td>
		<td>是否static</td>
	</tr>
	<tr>
		<td><font style="color:red;">ACC_FINAL</font></td>
		<td><font style="color:blue;">0x0010</font></td>
		<td>0000 0000 0001 0000</td>
		<td>是否final</td>
	</tr>
	<tr>
		<td><font style="color:red;">ACC_SYNCHRONIZED</font></td>
		<td><font style="color:blue;">0x0020</font></td>
		<td>0000 0000 0010 0000</td>
		<td>是否synchronized</td>
	</tr>
	<tr>
		<td><font style="color:red;">ACC_BRIDGE</font></td>
		<td><font style="color:blue;">0x0040</font></td>
		<td>0000 0000 0100 0000</td>
		<td>是否由编译器产生的桥接方法</td>
	</tr>
	<tr>
		<td><font style="color:red;">ACC_VARARGS</font></td>
		<td><font style="color:blue;">0x0080</font></td>
		<td>0000 0000 1000 0000</td>
		<td>方法是否接受不定参数</td>
	</tr>
	<tr>
		<td><font style="color:red;">ACC_NATIVE</font></td>
		<td><font style="color:blue;">0x0100</font></td>
		<td>0000 0001 0000 0000</td>
		<td>是否native</td>
	</tr>
	<tr>
		<td><font style="color:red;">ACC_ABSTRACT</font></td>
		<td><font style="color:blue;">0x0400</font></td>
		<td>0000 0100 0000 0000</td>
		<td>是否abstract</td>
	</tr>
	<tr>
		<td><font style="color:red;">ACC_STRICT</font></td>
		<td><font style="color:blue;">0x0800</font></td>
		<td>0000 1000 0000 0000</td>
		<td>是否strictfp</td>
	</tr>
		<tr>
		<td><font style="color:red;">ACC_SYNTHETIC</font></td>
		<td><font style="color:blue;">0x1000</font></td>
		<td>0001 0000 0000 0000</td>
		<td>是否由编译器自动产生</td>
	</tr>
</table>

#### __7.属性表集合__
属性表集合（attribute\_info）在前边大部分分析中一直没有遇到，直到TestClass中的方法1中才第一次出现attribute\_info的结构。它用于在Class文件、字段表、方发表中携带自己的属性表集合，以用于描述某些场景的专有信息。先看看下边

__*JVM虚拟机规范预定的属性*__

<table style="border-collapse:collapse;font-size:12px;">
	<tr style="font-style:italic">
		<td style="width:100px;">属性名称</td>
		<td>使用位置</td>
		<td>含义</td>
	</tr>
	<tr>
		<td><font style="color:blue;">Code</font></td>
		<td>方法表</td>
		<td>Java代码编译成的字节码指令</td>
	</tr>
	<tr>
		<td><font style="color:blue;">ConstantValue</font></td>
		<td>字段表</td>
		<td>final关键字定义的常量值</td>
	</tr>
	<tr>
		<td><font style="color:blue;">Deprecated</font></td>
		<td>类、方法表、字段表</td>
		<td>被声明为deprecated的方法和字段</td>
	</tr>
	<tr>
		<td><font style="color:blue;">Exceptions</font></td>
		<td>方法表</td>
		<td>方法抛出的异常</td>
	</tr>
	<tr>
		<td><font style="color:blue;">InnerClasses</font></td>
		<td>类文件</td>
		<td>内部类列表</td>
	</tr>
	<tr>
		<td><font style="color:blue;">LineNumberTable</font></td>
		<td>Code属性</td>
		<td>Java源码的行号与字节码指令的对应关系</td>
	</tr>
	<tr>
		<td><font style="color:blue;">LocalVariableTable</font></td>
		<td>Code属性</td>
		<td>方法的局部变量描述</td>
	</tr>
	<tr>
		<td><font style="color:blue;">SourceFile</font></td>
		<td>类文件</td>
		<td>源文件名称</td>
	</tr>
	<tr>
		<td><font style="color:blue;">Synthetic</font></td>
		<td>类、方法表、字段表</td>
		<td>标识方法或字段为编译器自动生成的</td>
	</tr>
</table>

##### __7.1.Code属性__
__*属性表结构*__<br/>
对于每个属性，名称需要从常量池中引用一个<font style="color:red">CONSTANT\_Utf8\_info</font>类型的常量来表示，而属性值的结构是完全自定义的，但符合规则的属性表应该满足下边的表结构。

<table style="border-collapse:collapse;font-size:12px;">
	<tr style="font-style:italic">
		<td style="width:100px;">类型</td>
		<td>名称</td>
		<td>数量</td>
	</tr>
	<tr>
		<td><font style="color:blue;">u2</font></td>
		<td><font style="color:red;">attribute_name_index</font></td>
		<td>1</td>
	</tr>
	<tr>
		<td><font style="color:blue;">u2</font></td>
		<td><font style="color:red;">attribute_length</font></td>
		<td>1</td>
	</tr>
	<tr>
		<td><font style="color:blue;">u2</font></td>
		<td><font style="color:red;">info</font></td>
		<td>attribute_length</td>
	</tr>
</table>

__*Code属性表结构*__<br/>
Java程序方法体里面的代码经过javac编译器处理过后将最终字节码存储在Code属性内。<font style="color:red;">*：抽象类或接口中的方法不存在Code属性。</font><br/>
Code属性的数据结构如下：
<table style="border-collapse:collapse;font-size:12px;">
	<tr style="font-style:italic">
		<td style="width:100px;">类型</td>
		<td>名称</td>
		<td>数量</td>
	</tr>
	<tr>
		<td><font style="color:blue;">u2</font></td>
		<td><font style="color:red;">attribute_name_index</font></td>
		<td>1</td>
	</tr>
	<tr>
		<td><font style="color:blue;">u4</font></td>
		<td><font style="color:red;">attribute_length</font></td>
		<td>1</td>
	</tr>
	<tr>
		<td><font style="color:blue;">u2</font></td>
		<td><font style="color:red;">max_stack</font></td>
		<td>1</td>
	</tr>
	<tr>
		<td><font style="color:blue;">u2</font></td>
		<td><font style="color:red;">max_locals</font></td>
		<td>1</td>
	</tr>
	<tr>
		<td><font style="color:blue;">u4</font></td>
		<td><font style="color:red;">code_length</font></td>
		<td>1</td>
	</tr>
	<tr>
		<td><font style="color:blue;">u1</font></td>
		<td><font style="color:red;">code</font></td>
		<td>code_length</td>
	</tr>
	<tr>
		<td><font style="color:blue;">u2</font></td>
		<td><font style="color:red;">exception_table_length</font></td>
		<td>1</td>
	</tr>
	<tr>
		<td><font style="color:blue;">exception_info</font></td>
		<td><font style="color:red;">exception_table</font></td>
		<td>exception_table_length</td>
	</tr>
	<tr>
		<td><font style="color:blue;">u2</font></td>
		<td><font style="color:red;">attributes_count</font></td>
		<td>1</td>
	</tr>
	<tr>
		<td><font style="color:blue;">attribute_info</font></td>
		<td><font style="color:red;">attributes</font></td>
		<td>attributes_count</td>
	</tr>
</table>

* attribute\_name\_index：指向<font style="color:red">CONSTANT\_Utf8\_info</font>类型常量的索引，常量值固定为“`Code`”，即属性的属性名为固定值，长度为4；
* attribute\_length：表示该属性的长度，属性长度 = 属性表长度 - 6字节（attribute\_name\_index占2字节，attribute_length占4字节）；
* max\_stack：操作数栈（Operand Stack）最大深度，任何时候操作数栈都不会超过这个深度，JVM根据这个值分配栈帧（Frame）；
* max\_locals：局部变量表所需的存储空间，max_locals单位是Slot，Slot是JVM为局部变量分配内存所使用的最小单位；<br/>
1）对于byte、char、float、int、short、boolean、reference、returnAddress长度不超过32位的数据类型，每个局部变量占用1个Slot；<br/>
2）而double和long这两种64位的数据类型则需要占用2个Slot存放
3）另外方法参数（包括实例方法中隐藏参数“this”）、显示异常处理器参数（Exception Handler Parameter，即try-catch语句中catch定义的异常）、方法中定义的局部变量也需要使用局部变量表来存。<font style="color:red">(Slot中存储的变量是可以重用的，max\_locals的大小并不是Slot之和。)</font>
* code\_length和code：存储了Java源程序编译后生成的字节码指令，code\_length是字节码长度，code是用于存储字节码指令的字节流；每一个字节码指令是u1类型，范围从0x00 ~ 0xFF，包含256条字节容量，而JVM中只有200条编码值对应。code\_length是u4类型，理论上最大值可以到（2^32 - 1），但JVM中限制了一个方法不允许超过65535条字节码指令，否则javac会拒绝编译；

Class中核心的两部分：

1. Code：代码区，方法体里的Java代码；
2. Metadata：元数据，类、字段、方法定义以及其他部分信息；

##### __7.2.Exception属性__
__*异常表结构*__<br/>
<table style="border-collapse:collapse;font-size:12px;">
	<tr style="font-style:italic">
		<td style="width:100px;">类型</td>
		<td>名称</td>
		<td>数量</td>
	</tr>
	<tr>
		<td><font style="color:blue;">u2</font></td>
		<td><font style="color:red;">start_pc</font></td>
		<td>1</td>
	</tr>
	<tr>
		<td><font style="color:blue;">u2</font></td>
		<td><font style="color:red;">end_pc</font></td>
		<td>1</td>
	</tr>
	<tr>
		<td><font style="color:blue;">u2</font></td>
		<td><font style="color:red;">handler_pc</font></td>
		<td>1</td>
	</tr>
	<tr>
		<td><font style="color:blue;">u2</font></td>
		<td><font style="color:red;">catch_type</font></td>
		<td>1</td>
	</tr>
</table>

他的基本含义是：如果字节码从start\_pc行（这里的行非源代码行）到第end\_pc行（不包含end\_pc行）之间出现了类型为catch\_type或其子类异常，则转到第handler\_pc行继续处理，当catch\_type为0时，代表任何异常情况都需要转到handler\_pc行处进行处理。

__*Exception属性结构*__<br/>
和前边的异常表结构不同，这里的Exception属性表是在方法中和Code属性平级，它的作用是列举出方法中可能抛出的受检查异常（Checked Exception），也就是方法描述时在throws关键字后边列举的异常。<font style="color:red;">*：只针对Checked Exception</font>
<table style="border-collapse:collapse;font-size:12px;">
	<tr style="font-style:italic">
		<td style="width:100px;">类型</td>
		<td>名称</td>
		<td>数量</td>
	</tr>
	<tr>
		<td><font style="color:blue;">u2</font></td>
		<td><font style="color:red;">attribute_name_index</font></td>
		<td>1</td>
	</tr>
	<tr>
		<td><font style="color:blue;">u4</font></td>
		<td><font style="color:red;">attribute_length</font></td>
		<td>1</td>
	</tr>
	<tr>
		<td><font style="color:blue;">u2</font></td>
		<td><font style="color:red;">number_of_exceptions</font></td>
		<td>1</td>
	</tr>
	<tr>
		<td><font style="color:blue;">u2</font></td>
		<td><font style="color:red;">exception_index_table</font></td>
		<td>number_of_exceptions</td>
	</tr>
</table>

##### __7.3.LineNumberTable属性__
LineNumberTable属性用于描述Java源代码行号和字节码行号（字节码的偏移量）之间的对应关系，它不是运行时必须属性，但默认会生成到Class文件中。也可以在javac中使用<font style="color:white;background-color:black">-g:none</font>或<font style="color:white;background-color:black">-g:lines</font>选项来取消或显示生成这一部分信息。<br/>
若不生成LineNumberTable的影响就是抛出Exception异常信息的时候不会在堆栈信息中显示行号，并且调试的时候无法按照源代码设置断点。

__*LineNumberTable属性结构*__<br/>
<table style="border-collapse:collapse;font-size:12px;">
	<tr style="font-style:italic">
		<td style="width:100px;">类型</td>
		<td>名称</td>
		<td>数量</td>
	</tr>
	<tr>
		<td><font style="color:blue;">u2</font></td>
		<td><font style="color:red;">attribute_name_index</font></td>
		<td>1</td>
	</tr>
	<tr>
		<td><font style="color:blue;">u4</font></td>
		<td><font style="color:red;">attribute_length</font></td>
		<td>1</td>
	</tr>
	<tr>
		<td><font style="color:blue;">u2</font></td>
		<td><font style="color:red;">line_number_table_length</font></td>
		<td>1</td>
	</tr>
	<tr>
		<td><font style="color:blue;">line_number_info</font></td>
		<td><font style="color:red;">line_number_table</font></td>
		<td>line_number_table_length</td>
	</tr>
</table>
这里只有一点需要说明：`line_number_info = start_pc + line_number`，两个变量都是u2类型的变量

* start\_pc：表示字节码行号；
* line\_number：表示Java源代码行号；

##### __7.4.LocalVariableTable属性__
LocalVariableTable属性用于描述栈帧中局部变量表中的变量与Java源代码定义的变量之间的关系，但是这种关系并非运行时必须，默认也不会生成到Class文件中，可以通过javac中使用<font style="color:white;background-color:black">-g:none</font>或<font style="color:white;background-color:black">-g:vars</font>选项取消或者生成这项信息。<br/>
如果没有这项信息，最大的影响就是其他人使用这个方法时，所有参数名会丢失，IDE可能使用arg0、arg1占位符替代原来的参数，这对程序运行没有影响，但会给写代码带来很大的不方便。

__*LocalVariableTable属性结构*__<br/>
<table style="border-collapse:collapse;font-size:12px;">
	<tr style="font-style:italic">
		<td style="width:100px;">类型</td>
		<td>名称</td>
		<td>数量</td>
	</tr>
	<tr>
		<td><font style="color:blue;">u2</font></td>
		<td><font style="color:red;">attribute_name_index</font></td>
		<td>1</td>
	</tr>
	<tr>
		<td><font style="color:blue;">u4</font></td>
		<td><font style="color:red;">attribute_length</font></td>
		<td>1</td>
	</tr>
	<tr>
		<td><font style="color:blue;">u2</font></td>
		<td><font style="color:red;">local_variable_table_length</font></td>
		<td>1</td>
	</tr>
	<tr>
		<td><font style="color:blue;">local_veriable_info</font></td>
		<td><font style="color:red;">local_variable_table</font></td>
		<td>local_variable_table_length</td>
	</tr>
</table>

*：这里的特殊变量是local\_variable\_info类型，它描述了一个栈帧与源代码中局部变量的关联，有单独的结构。

__*local\_variable\_info项目结构*__<br/>
<table style="border-collapse:collapse;font-size:12px;">
	<tr style="font-style:italic">
		<td style="width:100px;">类型</td>
		<td>名称</td>
		<td>数量</td>
	</tr>
	<tr>
		<td><font style="color:blue;">u2</font></td>
		<td><font style="color:red;">start_pc</font></td>
		<td>1</td>
	</tr>
	<tr>
		<td><font style="color:blue;">u2</font></td>
		<td><font style="color:red;">length</font></td>
		<td>1</td>
	</tr>
	<tr>
		<td><font style="color:blue;">u2</font></td>
		<td><font style="color:red;">name_index</font></td>
		<td>1</td>
	</tr>
	<tr>
		<td><font style="color:blue;">u2</font></td>
		<td><font style="color:red;">descriptor_index</font></td>
		<td>1</td>
	</tr>
	<tr>
		<td><font style="color:blue;">u2</font></td>
		<td><font style="color:red;">index</font></td>
		<td>1</td>
	</tr>
</table>
解释一下五个属性：

* start\_pc：表示这个局部变量的生命周期开始的字节码偏移量；
* length：表示这个局部变量的作用范围覆盖长度，和start\_pc一起就表示局部变量在字节码中的作用范围；
* name\_index：局部变量的名称对应常量池的符号引用；
* descriptor\_index：局部变量的类型对应的常量池的符号引用；
* index：局部变量在栈帧局部变量中Slot的位置，如果数据类型是long或double（64bit），Slot为index和index + 1两个位置；

<font style="color:red">*JDK 1.5引入了泛型之后，LocalVeriableTable属性添加了一个“姐妹属性”：LocalVariableTypeTable，这个属性的结构和LocalVariableTable相似，仅仅把记录的字段描述符的descriptor\_index替换成字段特征签名（Signature）。*</font>

##### __7.5.SourceFile属性__
SourceFile属性主要记录生成这个Class文件的源代码名称，也属于可选属性，可以使用javac的<font style="color:white;background-color:black">-g:none</font>或<font style="color:white;background-color:black">-g:source</font>选项来关闭或要求生成这些信息。

__*SourceFile结构*__<br/>
<table style="border-collapse:collapse;font-size:12px;">
	<tr style="font-style:italic">
		<td style="width:100px;">类型</td>
		<td>名称</td>
		<td>数量</td>
	</tr>
	<tr>
		<td><font style="color:blue;">u2</font></td>
		<td><font style="color:red;">attribute_name_index</font></td>
		<td>1</td>
	</tr>
	<tr>
		<td><font style="color:blue;">u4</font></td>
		<td><font style="color:red;">attribute_length</font></td>
		<td>1</td>
	</tr>
	<tr>
		<td><font style="color:blue;">u2</font></td>
		<td><font style="color:red;">sourcefile_index</font></td>
		<td>1</td>
	</tr>
</table>

##### __7.6.ConstantValue属性__
字节码中的ConstantValue的主要作用是虚拟机自动为静态变量赋值，只有被修饰了static关键字的变量（类变量）才可以使用这项属性。JVM对static类变量的赋值方式有两种：

* 在类构造器&lt;cinit&gt;中进行——在ConstantValue属性基础之上如果没有final修饰，并且不属于基本类型或java.lang.String，则使用&lt;cinit&gt;；
* 使用ConstantValue属性进行赋值——如果同时使用static和final，并且这个变量数据类型是8种基本类型或java.lang.String，则使用ConstantValue属性初始化；

__*ConstantValue结构*__<br/>
<table style="border-collapse:collapse;font-size:12px;">
	<tr style="font-style:italic">
		<td style="width:100px;">类型</td>
		<td>名称</td>
		<td>数量</td>
	</tr>
	<tr>
		<td><font style="color:blue;">u2</font></td>
		<td><font style="color:red;">attribute_name_index</font></td>
		<td>1</td>
	</tr>
	<tr>
		<td><font style="color:blue;">u4</font></td>
		<td><font style="color:red;">attribute_length</font></td>
		<td>1</td>
	</tr>
	<tr>
		<td><font style="color:blue;">u2</font></td>
		<td><font style="color:red;">constantvalue_index</font></td>
		<td>1</td>
	</tr>
</table>

##### __7.7.InnerClass属性__

__*InnerClass属性结构*__<br/>
<table style="border-collapse:collapse;font-size:12px;">
	<tr style="font-style:italic">
		<td style="width:100px;">类型</td>
		<td>名称</td>
		<td>数量</td>
	</tr>
	<tr>
		<td><font style="color:blue;">u2</font></td>
		<td><font style="color:red;">attribute_name_index</font></td>
		<td>1</td>
	</tr>
	<tr>
		<td><font style="color:blue;">u4</font></td>
		<td><font style="color:red;">attribute_length</font></td>
		<td>1</td>
	</tr>
	<tr>
		<td><font style="color:blue;">u2</font></td>
		<td><font style="color:red;">number_of_classes</font></td>
		<td>1</td>
	</tr>
	<tr>
		<td><font style="color:blue;">inner_classes_info</font></td>
		<td><font style="color:red;">inner_classes</font></td>
		<td>number_of_classes</td>
	</tr>
</table>

__*inner\_classes\_info表结构*__<br/>
<table style="border-collapse:collapse;font-size:12px;">
	<tr style="font-style:italic">
		<td style="width:100px;">类型</td>
		<td>名称</td>
		<td>数量</td>
	</tr>
	<tr>
		<td><font style="color:blue;">u2</font></td>
		<td><font style="color:red;">inner_class_info_index</font></td>
		<td>1</td>
	</tr>
	<tr>
		<td><font style="color:blue;">u2</font></td>
		<td><font style="color:red;">outer_class_info_index</font></td>
		<td>1</td>
	</tr>
	<tr>
		<td><font style="color:blue;">u2</font></td>
		<td><font style="color:red;">inner_name_index</font></td>
		<td>1</td>
	</tr>
	<tr>
		<td><font style="color:blue;">u2</font></td>
		<td><font style="color:red;">inner_class_access_flags</font></td>
		<td>1</td>
	</tr>
</table>

* inner\_name\_index：如果是匿名内部类，则这个值为0

__*inner\_class\_access\_flags标志*__<br/>
<table style="border-collapse:collapse;font-size:12px;">
	<tr style="font-style:italic">
		<td style="width:120px;">标志名称</td>
		<td>标志值</td>
		<td>二进制值</td>
		<td>含义</td>
	</tr>
	<tr>
		<td><font style="color:red;">ACC_PUBLIC</font></td>
		<td><font style="color:blue;">0x0001</font></td>
		<td>0000 0000 0000 0001</td>
		<td>是否public</td>
	</tr>
	<tr>
		<td><font style="color:red;">ACC_PRIVATE</font></td>
		<td><font style="color:blue;">0x0002</font></td>
		<td>0000 0000 0000 0010</td>
		<td>是否private</td>
	</tr>
	<tr>
		<td><font style="color:red;">ACC_PROTECTED</font></td>
		<td><font style="color:blue;">0x0004</font></td>
		<td>0000 0000 0000 0100</td>
		<td>是否protected</td>
	</tr>
	<tr>
		<td><font style="color:red;">ACC_STATIC</font></td>
		<td><font style="color:blue;">0x0008</font></td>
		<td>0000 0000 0000 1000</td>
		<td>是否static</td>
	</tr>
	<tr>
		<td><font style="color:red;">ACC_FINAL</font></td>
		<td><font style="color:blue;">0x0010</font></td>
		<td>0000 0000 0001 0000</td>
		<td>是否final</td>
	</tr>
	<tr>
		<td><font style="color:red;">ACC_SYNCHRONIZED</font></td>
		<td><font style="color:blue;">0x0020</font></td>
		<td>0000 0000 0010 0000</td>
		<td>是否synchronized</td>
	</tr>
	<tr>
		<td><font style="color:red;">ACC_ABSTRACT</font></td>
		<td><font style="color:blue;">0x0400</font></td>
		<td>0000 0100 0000 0000</td>
		<td>是否abstract</td>
	</tr>
	<tr>
		<td><font style="color:red;">ACC_SYNTHETIC</font></td>
		<td><font style="color:blue;">0x1000</font></td>
		<td>0001 0000 0000 0000</td>
		<td>是否由编译器自动产生</td>
	</tr>
		<tr>
		<td><font style="color:red;">ACC_ANNOTATION</font></td>
		<td><font style="color:blue;">0x2000</font></td>
		<td>0010 0000 0000 0000</td>
		<td>（JDK 1.5之后定义）标识这是一个注解</td>
	</tr>
	<tr>
		<td><font style="color:red;">ACC_ENUM</font></td>
		<td><font style="color:blue;">0x4000</font></td>
		<td>0100 0000 0000 0000</td>
		<td>（JDK 1.5之后定义）标识这是一个枚举</td>
	</tr>
</table>

##### __7.8.Deprecated, Synthetic属性__

Deprecated和Synthetic两个属性都是boolean标记，只存在有和没有的区别，没有属性值的概念，它们结构很简单
<table style="border-collapse:collapse;font-size:12px;">
	<tr style="font-style:italic">
		<td style="width:100px;">类型</td>
		<td>名称</td>
		<td>数量</td>
	</tr>
	<tr>
		<td><font style="color:blue;">u2</font></td>
		<td><font style="color:red;">attribute_name_index</font></td>
		<td>1</td>
	</tr>
	<tr>
		<td><font style="color:blue;">u4</font></td>
		<td><font style="color:red;">attribute_length</font></td>
		<td>1</td>
	</tr>
</table>

#### __8.JDK 1.5和JDK 1.6中的新属性__

<table style="border-collapse:collapse;font-size:12px;">
	<tr style="font-style:italic">
		<td style="width:160px;">属性名称</td>
		<td style="width:140px;">使用位置</td>
		<td>含义</td>
	</tr>
	<tr>
		<td><font style="color:blue;">StackMapTable</font></td>
		<td>Code属性</td>
		<td>JDK 1.6中添加，为了加快Class文件校验，把类型校验时需要用的相关信息直接写入class文件，以前这些信息通过代码数据流分析得到。</td>
	</tr>
	<tr>
		<td><font style="color:blue;">EnclosingMethod</font></td>
		<td>类</td>
		<td>JDK 1.5中添加的属性，当一个类为局部类或匿名类时，可通过此属性声明其访问范围。</td>
	</tr>
	<tr>
		<td><font style="color:blue;">Signature</font></td>
		<td>类、方法表、字段表</td>
		<td>JDK 1.5中添加的属性，存储类、方法、字段的特征签名。JDK 1.5引入泛型是Java语言的进步，虽然使用了类型擦除避免在字节码级别产生冲突，但元数据中的泛型信息需要保留，这种情况下描述符无法精确描述泛型信息，所以添加这个特征签名属性。</td>
	</tr>
	<tr>
		<td><font style="color:blue;">SourceDebugExtension</font></td>
		<td>类</td>
		<td>JDK 1.6中添加的属性，SourceDebugExtension属性用于存储额外调试信息，譬如JSP调试无法通过Java堆栈定位JSP的行号，JSR-45中为了非Java语言编写却需要编译成字节码运行在JVM中的程序提供了可进行调试的标准机制，使用SourceDebugExtension可存储这个标准新加入的调试信息。</td>
	</tr>
	<tr>
		<td><font style="color:blue;">LocalVariableTypeTable</font></td>
		<td>类</td>
		<td>JDK 1.5中添加的属性，使用特征签名代替描述符，为了引入泛型语法之后能描述泛型参数化类型而添加。</td>
	</tr>
	<tr>
		<td><font style="color:blue;">RuntimeVisibleAnnotations</font></td>
		<td>类、方法表、字段表</td>
		<td>JDK 1.5添加的属性，为动态注解提供支持，RuntimeVisibleAnnoations属性用于指明那些注解是运行时可见的。</td>
	</tr>
	<tr>
		<td><font style="color:blue;">RuntimeInvisibleAnnotations</font></td>
		<td>类、方法表、字段表</td>
		<td>JDK 1.5添加的属性，作用和上边作用刚好相反。</td>
	</tr>
	<tr>
		<td><font style="color:blue;">RuntimeVisibleParameterAnnotations</font></td>
		<td>方法表</td>
		<td>JDK 1.5添加的属性，作用和RuntimeVisiableAnnotations类似，只不过作用对象是参数。</td>
	</tr>
	<tr>
		<td><font style="color:blue;">RuntimeInvisibleParameterAnnotations</font></td>
		<td>方法表</td>
		<td>JDK 1.5添加的属性，不解释。</td>
	</tr>
	<tr>
		<td><font style="color:blue;">AnnotationDefault</font></td>
		<td>方法表</td>
		<td>JDK 1.5添加的属性，用于记录注解类元素的默认值</td>
	</tr>
</table>

<hr/>

### __解析`com.sco.core.TestClass`的字节码__

#### __1.魔数段（magic）__

	CA FE BA BE

Class字节码文件的头四个字节称为魔数（Magic Number)，唯一的作用是用于确定这个文件是否为一个虚拟机可接受的Class文件，Java字节码文件的魔数段是固定的，就是“咖啡宝贝”。

#### __2.Class文件版本（minor\_version major\_version）__

	00 00 00 34
	
紧跟魔数的4个字节是Class文件的版本号，第5和第6字节是次版本号（Minor Version，这里是`00 00`），第7和第8字节是主版本号（Major Version，这里是`00 34`），`34`是十六进制，对应十进制的52，即JDK 1.8的字节码。参考2.2章节的版本号详细内容，JDK版本从45开始到52，低版本的JVM是不能执行高版本的字节码的，范围是Version.0到Version.65535，比如JDK 1.7可执行的是51.0 ~ 51.65535。

#### __3.常量池（constant\_pool）__

##### *3.1.常量池入口*

	00 24
	
常量池入口是一个u2类型的数据，表示常量池容量计数（constant\_pool\_count），从1开始计数，24是十六进制，十进制为36，则表示常量池中有<font style="color:red">35项</font>常量，索引为1 ~ 35，索引0的位置为预留，可表示“不引用任何一个常量池项目”。<font style="color:red">只有常量池的容量计数是从1开始！！</font>

##### *3.2.常量池内容*

索引值和常量的标号对应，从1 ~ 35总共35个常量<br/>
常量1：

	0A 00 07 00 14			// java/lang/Object."<init>":()V

* `0A`——tag值为10，表示第一个常量类型是<font style="color:red">CONSTANT\_Methodref\_info</font>；
* `00 07`——<font style="color:blue">#7</font> 声明当前方法类描述符索引值为7；
* `00 14`——<font style="color:blue">#20</font> 当前方法的名称和类型索引值为20；

常量2：

	09 00 06 00 15			// com/sco/core/TestClass.age:I

* `09`——tag值为9，类型为<font style="color:red">CONSTANT\_Fieldref\_info</font>；
* `00 06`——<font style="color:blue">#6</font> 声明当前方法类描述符索引值为6；
* `00 15`——<font style="color:blue">#21</font> 字段描述符的名称和类型索引值为21；

常量3：

	09 00 06 00 16			// com/sco/core/TestClass.name:Ljava/lang/String;

* `09`——tag值为9，类型为<font style="color:red">CONSTANT\_Fieldref\_info</font>；
* `00 06`——<font style="color:blue">#6</font> 声明当前方法类描述符索引值为6；
* `00 16`——<font style="color:blue">#22</font> 字段描述符的名称和类型索引值为22；

<font style="color:red">*除了索引值，和第二个常量的其他内容都一致，也属于字段定义信息。*</font>

常量4：
	
	09 00 17 00 18			// java/lang/System.out:Ljava/io/PrintStream;

* `09`——tag值为9，类型为<font style="color:red">CONSTANT\_Fieldref\_info</font>；
* `00 17`——<font style="color:blue">#23</font> 声明当前方法类型描述符索引为23；
* `00 18`——<font style="color:blue">#24</font> 字段描述符的名称和类型索引值为24；

常量5：

	0A 00 19 00 1A			// java/io/PrintStream.out:Ljava/io/PrintStream;

* `0A`——tag值为10，类型为<font style="color:red">CONSTANT\_Methodref\_info</font>；
* `00 19`——<font style="color:blue">#25</font> 声明当前方法类描述符索引值为25；
* `00 1A`——<font style="color:blue">#26</font> 当前方法的名称和类型索引值为26；

常量6：

	07 00 1B				// com/sco/core/TestClass

* `07`——tag值为7，类型为<font style="color:red">CONSTANT\_Class\_info</font>；
* `00 1B`——<font style="color:blue">#27</font> 类型为“类或接口符号引用”，所以全限定名常量索引为27；

常量7：

	07 00 1C				// java/lang/Object

* `07`——tag值为7，类型为<font style="color:red">CONSTANT\_Class\_info</font>；
* `00 1C`——<font style="color:blue">#28</font> 类型为“类或接口符号引用”，所以全限定名常量索引为28；

常量8：

	01 00 03 61 67 65

* `01`——tag值为1，类型为<font style="color:red">CONSTANT\_Utf8\_info</font>；
* `00 03`——这个UTF-8编码的常量字符串长度为3，也就是说随后3个字节表示这个字符串常量；
* `61 67 65`——随后3个字节分别表示（字符串“age”）<br/><font style="color:blue">61 -> 97 -> a <br/>67 -> 103 -> g <br/>65 -> 101 -> e</font><br/><font style="color:blue">age</font>

常量9：

	01 00 01 49

* `01`——tag值为1，类型为<font style="color:red">CONSTANT\_Utf8\_info</font>；
* `00 01`——这个UTF-8编码的常量字符串长度为1；
* `49`——随后1个字节表示<br/><font style="color:blue">49 -> 73 -> I</font><br/><font style="color:blue">I</font>

常量10：

	01 00 04 6E 61 6D 65

* `01`——tag值为1，类型为<font style="color:red">CONSTANT\_Utf8\_info</font>；
* `00 04`——这个UTF-8编码的常量字符串长度为4；
* `6E 61 6D 65`——随后四个字节表示（字符串“name”）<br/><font style="color:blue">6E -> 110 -> n <br/>61 -> 97 -> a <br/>6D -> 109 -> m <br/>65 -> 101 -> e</font><br/><font style="color:blue">name</font>

常量11：

	01 00 12 4C 6A 61 76 61 2F 6C 61 6E 67 2F 53 74 72 69 6E 67 3B

* `01`——tag值为1，类型为<font style="color:red">CONSTANT\_Utf8\_info</font>；
* `00 12`——这个UTF-8编码的常量字符串长度为18；
* `4C 6A 61 76 61 2F 6C 61 6E 67 2F 53 74 72 69 6E 67 3B`——18个字节的字符串，对应：<font style="color:blue">Ljava/lang/String;</font>

常量12：

	01 00 06 3C 69 6E 69 74 3E

* `01`——tag值为1，类型为<font style="color:red">CONSTANT\_Utf8\_info</font>；
* `00 06`——这个UTF-8编码的常量字符串长度为6；
* `3C 69 6E 69 74 3E`——6个字节的字符串，对应：<font style="color:blue">&lt;init&gt;</font>

常量13：

	01 00 16 28 4C 6A 61 76 61 2F 6C 61 6E 67 2F 53 74 72 69 6E 67 3B 49 29 56

* `01`——tag值为1，类型为<font style="color:red">CONSTANT\_Utf8\_info</font>；
* `00 16`——这个UTF-8编码的常量字符串长度为22；
* `28 4C 6A 61 76 61 2F 6C 61 6E 67 2F 53 74 72 69 6E 67 3B 49 29 56`——22个字节的字符串，对应：<font style="color:blue">(Ljava/lang/String;I)V</font>

常量14：

	01 00 04 43 6F 64 65

* `01`——tag值为1，类型为<font style="color:red">CONSTANT\_Utf8\_info</font>；
* `00 04`——这个UTF-8编码的常量字符串长度为4；
* `43 6F 64 65`——4个字节的字符串，对应：<font style="color:blue">Code</font>

常量15：

	01 00 0F 4C 69 6E 65 4E 75 6D 62 65 72 54 61 62 6C 65

* `01`——tag值为1，类型为<font style="color:red">CONSTANT\_Utf8\_info</font>；
* `00 0F`——这个UTF-8编码的常量字符串长度为15；
* `4C 69 6E 65 4E 75 6D 62 65 72 54 61 62 6C 65`——15个字节的字符串，对应：<font style="color:blue">LineNumberTable</font>

常量16：

	01 00 03 69 6E 63

* `01`——tag值为1，类型为<font style="color:red">CONSTANT\_Utf8\_info</font>；
* `00 03`——这个UTF-8编码的常量字符串长度为3；
* `69 6E 63`——3个字节的字符串，对应：<font style="color:blue">inc</font>

常量17：

	01 00 03 28 29 49

* `01`——tag值为1，类型为<font style="color:red">CONSTANT\_Utf8\_info</font>；
* `00 03`——这个UTF-8编码的常量字符串长度为3；
* `28 29 49`——3个字节的字符串，对应：<font style="color:blue">()I</font>

常量18：

	01 00 0A 53 6F 75 72 63 65 46 69 6C 65

* `01`——tag值为1，类型为<font style="color:red">CONSTANT\_Utf8\_info</font>；
* `00 0A`——这个UTF-8编码的常量字符串长度为10；
* `53 6F 75 72 63 65 46 69 6C 65`——10字节的字符串，对应：<font style="color:blue">SourceFile</font>

常量19：

	01 00 0E 54 65 73 74 43 6C 61 73 73 2E 6A 61 76 61

* `01`——tag值为1，类型为<font style="color:red">CONSTANT\_Utf8\_info</font>；
* `00 0E`——这个UTF-8编码的常量字符串长度为14；
* `54 65 73 74 43 6C 61 73 73 2E 6A 61 76 61`——14字节的字符串，对应：<font style="color:blue">TestClass.java</font>

常量20：

	0C 00 0C 00 1D			// "lt;initgt;":()V

* `0C`——tag值为12，类型为<font style="color:red">CONSTANT\_NameAndType\_info</font>；
* `00 0C`——<font style="color:blue">#12</font> 该字段或方法名称常量索引为12；
* `00 1D`——<font style="color:blue">#29</font> 该字段或方法描述符常量索引为29；

常量21：

	0C 00 08 00 09			// age:I

* `0C`——tag值为12，类型为<font style="color:red">CONSTANT\_NameAndType\_info</font>；
* `00 08`——<font style="color:blue">#8</font> 该字段或方法名称常量索引为8；
* `00 09`——<font style="color:blue">#9</font> 该字段或方法描述符常量索引为9；

常量22：

	0C 00 0A 00 0B			// name:Ljava/lang/String; 

* `0C`——tag值为12，类型为<font style="color:red">CONSTANT\_NameAndType\_info</font>；
* `00 0A`——<font style="color:blue">#10</font> 该字段或方法名称常量索引为10；
* `00 0B`——<font style="color:blue">#11</font> 该字段或方法描述符常量索引为11；

常量23：

	07 00 1E				// java/lang/System 

* `07`——tag值为7，类型为<font style="color:red">CONSTANT\_Class\_info</font>；
* `00 1E`——<font style="color:blue">#30</font> 类型为“类或接口符号引用”，所以全限定名常量索引为30；

常量24：

	0C 00 1F 00 20			// out:Ljava/io/PrintStream; 

* `0C`——tag值为12，类型为<font style="color:red">CONSTANT\_NameAndType\_info</font>；
* `00 1F`——<font style="color:blue">#31</font> 该字段或方法名称常量索引为31；
* `00 20`——<font style="color:blue">#32</font> 该字段或方法描述符常量索引为32；

常量25：

	07 00 21				// java/io/PrintStream

* `07`——tag值为7，类型为<font style="color:red">CONSTANT\_Class\_info</font>；
* `00 21`——<font style="color:blue">#33</font> 类型为“类或接口符号引用”，所以全限定名常量索引为33；

常量26：

	0C 00 22 00 23			// println:(Ljava/lang/System;)V 

* `0C`——tag值为12，类型为<font style="color:red">CONSTANT\_NameAndType\_info</font>；
* `00 22`——<font style="color:blue">#34</font> 该字段或方法名称常量索引为34；
* `00 23`——<font style="color:blue">#35</font> 该字段或方法描述符常量索引为35；

常量27：

	01 00 16 63 6F 6D 2F 73 63 6F 2F 63 6F 72 65 2F 54 65 73 74 43 6C 61 73 73

* `01`——tag值为1，类型为<font style="color:red">CONSTANT\_Utf8\_info</font>；
* `00 16`——这个UTF-8编码的常量字符串长度为22；
* `63 6F 6D 2F 73 63 6F 2F 63 6F 72 65 2F 54 65 73 74 43 6C 61 73 73`——22字节的字符串，对应：<font style="color:blue">com/sco/core/TestClass</font>

常量28：

	01 00 10 6A 61 76 61 2F 6C 61 6E 67 2F 4F 62 6A 65 63 74

* `01`——tag值为1，类型为<font style="color:red">CONSTANT\_Utf8\_info</font>；
* `00 10`——这个UTF-8编码的常量字符串长度为16；
* `6A 61 76 61 2F 6C 61 6E 67 2F 4F 62 6A 65 63 74`——16字节的字符串，对应：<font style="color:blue">java/lang/Object</font>

常量29：

	01 00 03 28 29 56

* `01`——tag值为1，类型为<font style="color:red">CONSTANT\_Utf8\_info</font>；
* `00 03`——这个UTF-8编码的常量字符串长度为3；
* `28 29 56`——3个字节的字符串，对应：<font style="color:blue">()V</font>

常量30：

	01 00 10 6A 61 76 61 2F 6C 61 6E 67 2F 53 79 73 74 65 6D

* `01`——tag值为1，类型为<font style="color:red">CONSTANT\_Utf8\_info</font>；
* `00 10`——这个UTF-8编码的常量字符串长度为16；
* `6A 61 76 61 2F 6C 61 6E 67 2F 53 79 73 74 65 6D`——16字节的字符串，对应：<font style="color:blue">java/lang/System</font>

常量31：

	01 00 03 6F 75 74

* `01`——tag值为1，类型为<font style="color:red">CONSTANT\_Utf8\_info</font>；
* `00 03`——这个UTF-8编码的常量字符串长度为3；
* `6F 75 74`——3个字节的字符串，对应：<font style="color:blue">out</font>

常量32：

	01 00 15 4C 6A 61 76 61 2F 69 6F 2F 50 72 69 6E 74 53 74 72 65 61 6D 3B

* `01`——tag值为1，类型为<font style="color:red">CONSTANT\_Utf8\_info</font>；
* `00 15`——这个UTF-8编码的常量字符串长度为21；
* `4C 6A 61 76 61 2F 69 6F 2F 50 72 69 6E 74 53 74 72 65 61 6D 3B`——21个字节的字符串，对应：<font style="color:blue">Ljava/io/PrintStream;</font>

常量33：

	01 00 13 6A 61 76 61 2F 69 6F 2F 50 72 69 6E 74 53 74 72 65 61 6D

* `01`——tag值为1，类型为<font style="color:red">CONSTANT\_Utf8\_info</font>；
* `00 13`——这个UTF-8编码的常量字符串长度为19；
* `6A 61 76 61 2F 69 6F 2F 50 72 69 6E 74 53 74 72 65 61 6D`——19个字节的字符串，对应：<font style="color:blue">java/io/PrintStream</font>

常量34：
	
	01 00 07 70 72 69 6E 74 6C 6E

* `01`——tag值为1，类型为<font style="color:red">CONSTANT\_Utf8\_info</font>；
* `00 07`——这个UTF-8编码的常量字符串长度为7；
* `70 72 69 6E 74 6C 6E`——7个字节的字符串，对应：<font style="color:blue">println</font>

常量35 

	01 00 15 28 4C 6A 61 76 61 2F 6C 61 6E 67 2F 53 74 72 69 6E 67 3B 29 56

* `01`——tag值为1，类型为<font style="color:red">CONSTANT\_Utf8\_info</font>；
* `00 15`——这个UTF-8编码的常量字符串长度为21；
* `28 4C 6A 61 76 61 2F 6C 61 6E 67 2F 53 74 72 69 6E 67 3B 29 56`——21个字节的字符串，对应：<font style="color:blue">(L/java/lang/String;)V</font>

<font style="color:red">*上边列举了例子中35个常量的字节码内容，可仔细去对照Class字节文件内容看看常量池的定义信息。常量池的详细信息相对比较繁琐因为每一种常量类型都对应了自己的一种结构，对照上边的详细内容结构表可解析每一个常量的类型、长度、详细内容是什么。*</font>

#### __4.访问标志（access_flags）__
访问标志：

	00 21

该类的访问标志为：`0x0021 = 0x0020 | 0x0001 = ACC_SUPER | ACC_PUBLIC`

#### __5.类索引、父类索引、接口索引__

* 类索引：引用于确定这个类的全限定名；
* 父类索引：引用于确定这个类的父类的全限定名（因为Java语言不支持多继承，所有的类都继承于java.lang.Object，除了java.lang.Object类，所有类的父索引都不为0）；
* 接口索引集：接口索引的格式一般格式是：interfaces\_count （ u2 ) + interfaces ( u2 ) * n；（ n -- interfaces\_count ），这里interfaces\_count表示当前类继承了多少接口，是接口计数器，后边每一个u2类型的正数就是每一个接口的接口索引；

TestClass示例：

	00 06 00 07 00 00

* `00 06`：类索引为<font style="color:blue">#6</font>，值：#6 -> #27 -> com/sco/core/TestClass
* `00 07`：父类索引为<font style="color:blue">#7</font>，值：#7 -> #28 -> java/lang/Object
* `00 00`：因为TestClass类没有实现任何接口，所以接口索引集部分为`00 00`，并且紧随其后也没有任何字节描述

#### __6.字段表集合__
字段计数器：（当前类中有2个字段）

	00 02

字段1：

	00 82 00 08 00 09 00 00

* `00 82`：access\_flags = 0x0080 | 0x0002 = ACC_TRANSIENT | ACC_PRIVATE
* `00 08`：name\_index = <font style="color:blue;">#8</font>，#8 -> age
* `00 09`：descriptor\_index = <font style="color:blue;">#9</font>，#9 -> I
* `00 00`：attributes_count：值为0，因为值为0，所以之后自然没有attribute\_info部分

字段2：

	00 02 00 0A 00 0B 00 00

* `00 02`：access\_flags = 0x0002 = ACC_PRIVATE
* `00 0A`：name\_index = <font style="color:blue;">#10</font>，#10 -> name
* `00 0B`：descriptor\_index = <font style="color:blue;">#11</font>，#11 -> Ljava/lang/String;
* `00 00`：attributes_count：值为0，所以之后就没有attribute\_info部分

#### __7.方法表、Code属性__
方法计数器：（当前类中有2个方法）

	00 02

##### __7.1.第一个方法__
方法1（构造方法）：

	00 01 00 0C 00 0D 00 01

* `00 01`：access\_flags = 0x0001 = ACC\_PUBLIC
* `00 0C`：name\_index = <font style="color:blue;">#12</font>, #12 -> &lt;init&gt;
* `00 0D`：descriptor\_index = <font style="color:blue;">#13</font>，#13 -> (Ljava/lang/String;I)V
* `00 01`：attributes\_count：值为1，所以紧随其后的就是attribute\_info部分

方法1的Code（非指令部分）：

	00 0E 00 00 00 33 00 02 00 03 00 00 00 0F				// 非指令部分

* `00 0E`：attribute\_name\_index = <font style="color:blue;">#14</font>，#14 -> Code
* `00 00 00 33`：attribute\_length = 33 -> 51，所以整个属性表的长度为51 + 6 = 57字节长度
* `00 02`：max\_stack = 2
* `00 03`：max\_locals = 3
* `00 00 00 0F`：code\_length = 15

方法1的Code（指令部分）：

	2A B7 00 01 2A 1C B5 00 02 2A 2B B5 00 03 B1			// 指令部分

* `2A B7 00 01`<br/>`2A -> aload_0`：调用aload\_0指令将第一个Reference类型的本地变量推送至栈顶，存储在第0个Slot中；<br/>
`B7 00 01-> invokespecial #1`：调用超类构造方法、实例初始化方法、私有方法，invokespecial之后有一个u2类型的参数，对应&lt;init&gt;的符号引用<br/>
* `2A 1C B5 00 02`<br/>`2A -> aload_0`：调用aload\_0指令将第一个Reference类型的本地变量推送至栈顶<br/>
`1C -> iload_2`：调用iload\_2将第三个int整型本地变量推送到栈顶；<br/>
`B5 00 02 -> putfield #2`：调用putfield为指定的实例域赋值，00 02的常量为age的符号引用；<br/>
* `2A 2B B5 00 03`<br/>`2A -> aload_0`：调用aload\_0指令将第一个Reference类型的本地变量推送至栈顶<br/>
`2B -> aload_1`：调用aload\_1指令将第二个Reference类型的本地变量推送至栈顶；<br/>
`B5 00 03 -> putfield #3`：调用putfield为指定的实例域赋值，00 03的常量为name的符号引用；<br/>
* `B1`：最后一个B1指令为：`B1 -> return`表示当前方法返回void，到这里构造函数就调用完成了；

方法1的Exception：

	00 00  				// 该方法没有throws部分的定义

方法1的Attribute Count：

	00 01				// 方法1最后一部分有一个属性块

方法1的LineNumberTable：

	00 0F 00 00 00 12 00 04 
	00 00 00 07 00 04 00 08 00 09 00 09 00 0E 00 0A

* `00 0F`：attribute\_name\_index = <font style="color:blue;">#15</font>，#15 -> LineNumberTable
* `00 00 00 12`：attribute\_length = 14
* `00 04`：line\_number\_table\_length = 4，表示这个LineNumberTable中有4条记录
* `00 00 00 07 00 04 00 08 00 09 00 09 00 0E 00 0A`：Source File -> Byte Code<br/>
`00 00 00 07` -> Source File( 7 ) : Byte Code ( 0 )<br/>
`00 04 00 08` -> Source File( 8 ) : Byte Code ( 4 )<br/>
`00 09 00 09` -> Source File( 9 ) : Byte Code ( 9 )<br/>
`00 0E 00 0A` -> Source File( 14 ) : Byte Code ( 10 )

<font style="color:red">到这里构造函数的方法1部分的字节码就全部解析完了，接下来看看剩余部分的方法2的字节码。</font>

##### __7.2.第二个方法__
方法2：

	00 01 00 10 00 11 00 01

* `00 01`：access\_flags = 0x0001 = ACC\_PUBLIC
* `00 10`：name\_index = <font style="color:blue;">#16</font>, #16 -> inc
* `00 11`：descriptor\_index = <font style="color:blue;">#17</font>, #17 -> ()I
* `00 01`：attributes\_count：值为1，紧随其后就是attribute\_info

方法2的Code（非指令部分）：

	00 0E 00 00 00 2D 00 02 00 01 00 00 00 11

* `00 0E`：attribute\_name\_index = <font style="color:blue;">#14</font>，#14 -> Code
* `00 00 00 2D`：attribute\_length = 2D -> 45，所以整个属性表的长度为45 + 6 = 51字节长度
* `00 02`：max\_stack = 2
* `00 01`：max\_locals = 1
* `00 00 00 11`：code\_length = 17

方法2的Code（指令部分）：

	B2 00 04 2A B4 00 03 B6 00 05 2A B4 00 02 04 60 AC			// 指令部分

* `B2 00 04`<br/>`B2 00 04 -> getstatic #4`：获取指定类的静态域，并且压入到栈顶，这里#4表示指定类的符号引用，为：java/lang/System.out:Ljava/io/PrintStream;
* `2A B4 00 03`<br/>`2A -> aload_0`：调用aload\_0指令将第一个Reference类型的本地变量推送到栈顶<br/>
`B4 00 03 -> getfield #3`：获取指定类的实例域，并且将其压入到栈顶，#3的符号引用为：com/sco/core/TestClass.name:Ljava/lang/String;，即TestClass的实例变量name；
* `B6 00 05`<br/>`B6 00 05 -> invokevirtual #5`：调用实例方法，#5的符号引用为：java/io/PrintStream.println:(Ljava/lang/String;)V
* `2A B4 00 02`<br/>`2A -> aload_0`：调用aload\_0指令将第一个Reference类型的本地变量推送到栈顶<br/>
`B4 00 02 -> getfield #2`：获取指定类的实例域，并且将其压入到栈顶，#2的符号引用为：com/sco/core/TestClass.age:I
* `04 60 AC`<br/>`04 -> iconst_1`：将int类型的1推送到栈顶<br/>
`60 -> iadd`：将栈顶两个int类型的值相加，返回结果重新推送到栈顶<br/>
`AC -> ireturn`：从当前方法返回int值

方法2的Exception：

	00 00  				// 该方法没有throws部分的定义

方法2的Attribute Count：

	00 01				// 方法1最后一部分有一个属性块

方法2的LineNumberTable：

	00 0F 00 00 00 0A 00 02
	00 00 00 0D 00 0A 00 0E

* `00 0F`：attribute\_name\_index = <font style="color:blue;">#15</font>，#15 -> LineNumberTable
* `00 00 00 0A`：attribute\_length = 10
* `00 02`：line\_number\_table\_length = 2，表示这个LineNumberTable中有2条记录
* `00 00 00 0D 00 0A 00 0E`：Source File -> Byte Code<br/>
`00 00 00 0D` -> Source File( 13 ) : Byte Code ( 0 )<br/>
`00 0A 00 0E` -> Source File( 14 ) : Byte Code ( 10 )

#### __8.SourceFile属性__

	00 01 						// 方法区过后当前Class文件也会包含attribute属性信息，当前Class文件还有1个属性
	00 12 00 00 00 02 00 13

* `00 12`：attribute\_name\_index = <font style="color:blue;">#18</font>，#18 -> SourceFile
* `00 00 00 02`：attribute\_length = 2
* `00 13`：sourcefile\_index = <font style="color:blue;">#19</font>, #19 -> TestClass.java

<font style="color:red">*到这里com.sco.core.TestClass这个类的字节码文件就全部解析完成了。*</font>