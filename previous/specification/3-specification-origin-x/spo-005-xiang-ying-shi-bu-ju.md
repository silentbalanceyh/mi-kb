## 响应式布局

[SPO-004 页面布局说明](/specification/3-specification-origin-x/spo-004-ye-mian-bu-ju-shuo-ming.html)中提到的一个页面的片段如：

```json
{
    "span":16,
    "value":[
        "ee61e2d9-93e4-4478-971d-36e24f59723e",
        "a4031496-6460-4464-9d6f-e1fb20cd16d3"
    ]
}
```

Origin X中的响应式布局直接沿用Ant Design中的响应式处理：[https://ant.design/components/grid-cn/\#components-grid-demo-responsive](https://ant.design/components/grid-cn/#components-grid-demo-responsive)，通过对`<Col/>`进行特殊的响应式配置来实现响应式布局。

## 1. 响应式

| 属性名 | 说明 |
| :--- | :--- |
| xs | 480px &lt;= 宽度 &lt; 576px |
| sm | 576px &lt;= 宽度 &lt; 768px |
| md | 768px &lt;= 宽度 &lt; 992px |
| lg | 992px &lt;= 宽度 &lt; 1200px |
| xl | 1200px &lt;= 宽度 &lt; 1600px |
| xxl | 1600px &lt;= 宽度 |

这里的宽度表示屏幕的分辨率宽度，可以看看Ant Design中的变量定义：

```less
// Media queries breakpoints
// Extra small screen / phone
@screen-xs: 480px;
@screen-xs-min: @screen-xs;

// Small screen / tablet
@screen-sm: 576px;
@screen-sm-min: @screen-sm;

// Medium screen / desktop
@screen-md: 768px;
@screen-md-min: @screen-md;

// Large screen / wide desktop
@screen-lg: 992px;
@screen-lg-min: @screen-lg;

// Extra large screen / full hd
@screen-xl: 1200px;
@screen-xl-min: @screen-xl;

// Extra extra large screen / large descktop
@screen-xxl: 1600px;
@screen-xxl-min: @screen-xxl;
```

## 2. 响应式配置

下边是响应式布局的示例：

```json
{
    "page":{
        "layout":[
            "2ba7d8a5-c840-4bdf-9449-72c1407c5262",
            [
                {
                    "span":8,
                    "lg":6,
                    "value":[
                        "6066a32c-d0b4-4243-8e21-b495cf49b1d4"
                    ]
                },
                {
                    "span":16,
                    "lg":18,
                    "value":[
                        "b2fbb200-0371-4ca0-a174-080562613087"
                    ]
                }
            ]
        ]
    }
}
```



