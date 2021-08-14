# UBL Price说明

[Reference](http://www.oioubl.info/documents/en/en/Guidelines/OIOUBL_GUIDE_PRICES.pdf)

本文档翻译自引用PDF文件，用于描述UBL中的Price和Quantity的使用方法。

## 1. UBL Classes & Elements

在UBL的行级别，下边的元素用于描述价格和数量信息：

* OrderLine
* InvoiceLine

在OrderLine元素中，主要涉及的结构如下（OrderLine/LineItem）：

* Quantity
* LineExtensionAmount
* Delivery / Quantity
* Item / PackQuantity
* Item / PackSizeNumeric
* Price / PriceAmount
* Price / BaseQuantity
* Price / OrderableUnitFactorRate

```xml
<cbc:OrderLine>
    <cbc:LineItem>
        <cbc:Quantity unitCode="LTR">10</cbc:Quantity>
        <cbc:LineExtensionAmount currencyID="CNY">120</cbc:LineExtensionAmount>
        <cbc:Delivery>
            <cbc:Quantity unitCode="LTR">20</cbc:Quantity>
        </cbc:Delivery>
        <cbc:Item>
            <cbc:PackQuantity unitCode="EA">30</cbc:PackQuantity>                
            <cbc:PackSizeNumeric>40</cbc:PackSizeNumeric>
        </cbc:Item>
        <cbc:Price>
            <cbc:PriceAmount currencyID="CNY">1</cbc:PriceAmount>
            <cbc:BaseQuantity unitCode="EA">9</cbc:BaseQuantity>
            <cbc:OrderableUnitFactorRate>1</cbc:OrderableUnitFactoryRate>
        </cbc:Price>
    </cbc:LineItem>
</cbc:OrderLine>
```

在InvoiceLine元素中，主要涉及的结构如下（InvoiceLine）：

* InvoicedQuantity
* LineExtensionAmount
* Delivery / Quantity
* Price / PriceAmount
* Price / BaseQuantity
* Price / OrderableUnitFactorRate

```xml
<cac:InvoiceLine>
    <cbc:InvoicedQuantity unitCode="LTR">10</cbc:InvoicedQuantity>
    <cbc:LineExtensionAmount currencyID="CNY">120</cbc:LineExtensionAmount>
    <cac:Delivery>
        <cbc:Quantity unitCode="LTR">20</cbc:Quantity>
    </cac:Delivery>
    <cac:Item>
        <cbc:PackQuantity unitCode="EA">30</cbc:PackQuantity>
        <cbc:PackSizeNumeric>40</cbc:PackSizeNumeric>
    </cac:Item>
    <cac:Price>
        <cbc:PriceAmount currencyID="CNY">1</cbc:PriceAmount>
        <cbc:BaseQuantity unitCode="EA">9</cbc:BaseQuantity>
        <cbc:OrderableUnitFactorRate>1</cbc:OrderableUnitFactoryRate>
    </cac:Price>
</cac:InvoiceLine>
```

## 2. Element Explanations

| UK-name | Use | Remarks |
| :--- | :--- | :--- |
| Quantity | 1 | 和单位对应的数量信息 |
| Quantity@unitCode |  | 数量对应的单位代码 |
| PackQuantity | 0..1 | Item中打包的数量信息，这个信息在PackSizeNumeric中有描述。 |
| PackQuantity@unitCode |  | 指定单位代码 |
| PackSizeNumeric | 0..1 | 打包时一个包的数量信息 |
| PriceAmount | 1 | 基于BaseQuantity的最终金额信息，浮点格式 |
| PriceAmount@currencyID |  | 价格对应的货币代码 |
| BaseQuantity | 0..1 | 基数数量信息 |
| BaseQuantity@unitCode |  | 指定BaseQuantity的单位代码 |
| OrderableUnitFactorRate | 0..1 | 针对BaseQuantity的计算因子 |

_注意：BaseQuantity和OrderableUnitFactorRate应该不为空，并且给予默认值：BaseQuantity默认值为"1 EA"（Each），OrderableUnitFactorRate的默认值为1。_

## 3. Price和Quantity的关系

### 3.1. BaseQuantity

Price最终结果是通过计算得到了的，看看下边例子：

**Simple**

```xml
<cac:InvoiceLine>
    <cbc:InvoicedQuantity unitCode="BO">12</cbc:InvoicedQuantity>
    <cbc:LineExtensionTotalAmount currencyID="DKK">720.00</cbc:LineExtensionTotalAmount>
    <cac:Item>
        <cbc:Name>Red wine</cbc:Name>
        <cac:SellersItemIdentification>
            <cbc:ID>1234567</cbc:ID>
        </cac:SellersItemIdentification>
    </cac:Item>
    <cac:Price>
        <cbc:PriceAmount currencyID="DKK">60.00</cbc:PriceAmount>
        <cbc:BaseQuantity unitCode="BO">1</cbc:BaseQuantity>
        <cbc:OrderableUnitFactorRate>1</cbc:OrderableUnitFactorRate>
    </cac:Price>
</cac:InvoiceLine>
```

上边描述了InvoicedQuantity的unitCode = BO，值为12，则表示12 Bottles，总价格为DKK 720，实际上每一个单位（Bottle）的价格是DKK 60.00，每一个单位（OrderableUnitFactorRate）包含了1 Bottle，所以最终的Amount计算结果为DKK 720。

**Advanced**

```xml
<cac:InvoiceLine>
    <cbc:InvoicedQuantity unitCode="CS">1</cbc:InvoicedQuantity>
    <cbc:LineExtensionTotalAmount currencyID="DKK">720.00</cbc:LineExtensionTotalAmount>
    <cac:Item>
        <cbc:Name>Red wine</cbc:Name>
        <cac:SellersItemIdentification>
            <cbc:ID>1234567</cbc:ID>
        </cac:SellersItemIdentification>
    </cac:Item>
    <cac:Price>
        <cbc:PriceAmount currencyID="DKK">60.00</cbc:PriceAmount>
        <cbc:BaseQuantity unitCode="BO">1</cbc:BaseQuantity>
        <cbc:OrderableUnitFactorRate>12</cbc:OrderableUnitFactorRate>
    </cac:Price>
</cac:InvoiceLine>
```

上边描述了每一个单位（Bottle）价格为DKK 60.00，而每一个单元（OrderableUnitFactorRate）是12 Bottles。，最终结果和上边计算结果一致。

**Delivery Unit**

```xml
<cac:InvoiceLine>
    …
    <cac:Delivery>
        <cbc:Quantity unitCode="CS">1</cbc:Quantity>
    </cac:Delivery>
    …
</cac:InvoiceLine>
```

### 3.2. Orderable and Invoice Unit

**OrderableUnit**

该属性在OrderLine/LineItem中使用，示例如下：

```xml
<cac:OrderLine>
    …
    <cac:LineItem>
        <cbc:ID>1</cbc:ID>
        <cbc:Quantity unitCode="CS">1</cbc:Quantity>
        <cbc:LineExtensionAmount currencyID="DKK">720</cbc:LineExtensionAmount>
        <cac:Price>
            <cbc:PriceAmount currencyID="DKK">60.00</cbc:PriceAmount>
            <cbc:BaseQuantity unitCode="BO">1</cbc:BaseQuantity>
            <cbc:OrderableUnitFactorRate>12</cbc:OrderableUnitFactorRate>
        </cac:Price>
        <cac:Item>
            <cbc:Name>Red wine</cbc:Name>
            <cac:SellersItemIdentification>
                <cbc:ID>1234567</cbc:ID>
            </cac:SellersItemIdentification>
        </cac:Item>
    </cac:LineItem>
    …
<cac:OrderLine>
```

上述几种属性的直接关系如下（`LineExtensionAmount、Quantity、PriceAmount、BaseQuantity、OrderableUnitFactorRate`）：

```
BaseQuantity x OrderableUnitFactorRate = Quantity@unitCode中按照计量单位描述的信息
```

* BaseQuantity = 1 BO（Bottle）
* OrderableUnitFactorRate = 12
* Quantity@unitCode = CS（Case）

则表示1 case of 12 bottles：一件12瓶，最终计算结果为：

```
PriceAmount / BaseQuantity x ( BaseQuantity x OrderableUnitFactorRate ) = 订单中每一个单位的价格（单价）
PriceAmount x OrderableUnitFactorRate = 这个订单单位的价格（非单价）
```

* PriceAmount = DKK 60.00
* BaseQuantity = 1
* OrderableUnitFactorRate = 12

那么LineExtensionAmount的值应该是DKK 720.00，并且含义为12瓶一件，实际上这个价格是每一个订单单位的价格。

**InvoicedQuantity**

```xml
<cac:InvoiceLine>
    …
    <cbc:ID>1</cbc:ID>
    <cbc:InvoicedQuantity unitCode="BO">12</cbc:InvoicedQuantity>
    <cbc:LineExtensionAmount currencyID="DKK">720</cbc:LineExtensionAmount>
    <cac:Item>
        <cbc:Name>Red wine</cbc:Name>
        <cac:SellersItemIdentification>
            <cbc:ID>1234567</cbc:ID>
        </cac:SellersItemIdentification>
    </cac:Item>
    <cac:Price>
        <cbc:PriceAmount currencyID="DKK">60.00</cbc:PriceAmount>
        <cbc:BaseQuantity unitCode="BO">1</cbc:BaseQuantity>
        <cbc:OrderableUnitFactorRate>1</cbc:OrderableUnitFactorRate>
    </cac:Price>
    …
<cac:InvoiceLine>
```

上述例子中的计算为：

```
LineExtensionAmount = PriceAmount / BaseQuantity x (BaseQuantity x OrderableUnitFactorRate ) x InvoicedQuantity
```

**PackSizeNumeric**

和单位还相关的是另外两个元素PackQuantity和PackSizeNumeric，它的定义如下：

```xml
<cac:Item>
    …
    <cbc:PackQuantity unitCode="CS">1</cbc:PackQuantity>
    <cbc:PackSizeNumeric>12</cbc:PackSizeNumeric>
    …
<cac:Item>
```

计算含义为：

```
BaseQuantity x PackSizeNumeric = PackQuantity@unitCode中描述的单位计量
```

## 4. BaseQuantity -&gt; InvoicedQuantity

```xml
<cac:InvoiceLine>
    …
    <cbc:ID>1</cbc:ID>
    <cbc:InvoicedQuantity unitCode="BLL">1</cbc:InvoicedQuantity>
    <cbc:LineExtensionAmount currencyID="DKK">3600.00</cbc:LineExtensionAmount>
    <cac:Item>
        <cbc:Name>Let smøreolie</cbc:Name>
        <cac:SellersItemIdentification>
            <cbc:ID>11223344</cbc:ID>
        </cac:SellersItemIdentification>
    </cac:Item>
    <cac:Price>
        <cbc:PriceAmount currencyID="DKK">4800.00</cbc:PriceAmount>
        <cbc:BaseQuantity unitCode="LTR">1000</cbc:BaseQuantity>
        <cbc:OrderableUnitFactorRate>0.75</cbc:OrderableUnitFactorRate>
    </cac:Price>
    …
<cac:InvoiceLine>
```

原文解释：

The quantity unit code of "BLL" in InvoicedQuantity specifies that the invoiced quantity is a barrel. The supplier's base price \(PriceAmount\) is DKK 4800.00 for kilolitre \(the BaseQuantity\). Because the supplier sells the oil in barrels of 750 litres and not kilolitres, the supplier must specify the conversion factor \(OrderableUnitFactorRate\) that should be applied to convert the supplier's base quantity to the unit of 1 barrel. In this case, the OrderableUnitFactorRate is 0.75. \(1000 litres \* 0.75 = 750 litres ≈ 1 barrel\). The price of one barrel of oil is calculated by multiplying the supplier's base price \(PriceAmount\) with the OrderableUnitFactorRate, that is DKK 4800.00 \* 0.75 or DKK 3600.00 per barrel.

## 5. UBL中的unitCode和currencyID

CurrencyID

[https://en.wikipedia.org/wiki/ISO\_4217](https://en.wikipedia.org/wiki/ISO_4217)

UnitCode

[https://www.unece.org/fileadmin/DAM/cefact/recommendations/rec20/rec20\_rev3\_Annex2e.pdf](https://www.unece.org/fileadmin/DAM/cefact/recommendations/rec20/rec20_rev3_Annex2e.pdf)