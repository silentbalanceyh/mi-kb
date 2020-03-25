# 界面配置全Json示例

下边这份数据是页面[http://localhost:5000/ox/dp/person/zone](http://localhost:5000/ox/dp/person/zone)的配置：

```json
{
    "comment": {
        "content": "页面基础规范，用于描述节点的基本配置信息",
        "nodes": {
            "layout": "UI_LAYOUT 中的 uiData",
            "page": "UI_PAGE 中的 uiData 加上各种 Slice"
        }
    },
    "layout": {
    },
    "page": {
        "grid": [
            "618b8b98-90d3-4823-ab4d-736996bea904",
            "54740a4b-1257-4985-a8f5-d49ad76852d5"
        ],
        "container": null
    },
    "ajax": {
        "personal.circle.by.type": {
            "uri": "/api/personal/circle/type"
        },
        "personal.pager.zone": {
            "uri": "/api/personal/circle/search",
            "method": "POST",
            "query": {
                "projection": [],
                "pager": "1,5",
                "sorter": "createdAt=DESC",
                "criteria": {
                }
            }
        }
    },
    "control": {
        "618b8b98-90d3-4823-ab4d-736996bea904": {
            "container": {
                "name": "OxCard",
                "config": {
                    "header": false
                }
            },
            "component": {
                "name": "OxCount",
                "data": "personal.circle.by.type",
                "config": [
                    {
                        "key": "own",
                        "label": "我创建的圈子",
                        "value": ":own个"
                    },
                    {
                        "key": "consume",
                        "label": "我加入的圈子",
                        "value": ":consume个"
                    }
                ]
            }
        },
        "54740a4b-1257-4985-a8f5-d49ad76852d5": {
            "container": {
                "name": "OxCard",
                "config": {
                    "header": false
                }
            },
            "component": {
                "name": "OxContainer",
                "config": {
                    "grid": [
                        [
                            {
                                "span": 16,
                                "lg": 14,
                                "xxl": 16,
                                "value": [
                                    "f47ef525-b4b4-44cb-9870-478ba5ce71ff"
                                ]
                            },
                            {
                                "span": 4,
                                "lg": 5,
                                "xxl": 4,
                                "value": [
                                    "1e023544-3b19-4c4b-8dee-92617c0ae6c0"
                                ]
                            },
                            {
                                "span": 4,
                                "lg": 5,
                                "xxl": 4,
                                "value": [
                                    "98d5f0ed-274e-4919-b0de-3a9b432b60b8"
                                ]
                            }
                        ],
                        "1d43acbc-0079-4475-b274-d32f65b7ebe7"
                    ]
                }
            }
        },
        "f47ef525-b4b4-44cb-9870-478ba5ce71ff": {
            "component": {
                "name": "OxTitle",
                "config": {
                    "title": "我的圈子",
                    "style": {
                        "paddingLeft": 24,
                        "paddingTop": 8,
                        "paddingBottom": 8
                    }
                }
            }
        },
        "1e023544-3b19-4c4b-8dee-92617c0ae6c0": {
            "component": {
                "name": "OxActRadio",
                "event": [
                    {
                        "name": "onSearch",
                        "config": {
                            "target": "1d43acbc-0079-4475-b274-d32f65b7ebe7",
                            "query": {
                                "category": ":value"
                            }
                        }
                    }
                ],
                "config": {
                    "type": "BUTTON",
                    "style": {
                        "paddingTop": 8,
                        "paddingBottom": 8
                    },
                    "defaultValue": "__DELETE__",
                    "items": [
                        {
                            "label": "全部",
                            "key": "__DELETE__"
                        },
                        {
                            "label": "创建的",
                            "key": "OWNER"
                        },
                        {
                            "label": "加入的",
                            "key": "JOINED"
                        }
                    ]
                }
            }
        },
        "98d5f0ed-274e-4919-b0de-3a9b432b60b8": {
            "component": {
                "name": "OxActSearch",
                "event": [
                    {
                        "name": "onSearch",
                        "config": {
                            "target": "1d43acbc-0079-4475-b274-d32f65b7ebe7",
                            "query": {
                                "title,c": ":value"
                            }
                        }
                    }
                ],
                "config": {
                    "style": {
                        "paddingTop": 8,
                        "paddingBottom": 8,
                        "width": "80%"
                    },
                    "placeholder": "按标题搜索"
                }
            }
        },
        "1d43acbc-0079-4475-b274-d32f65b7ebe7": {
            "component": {
                "name": "OxListTask",
                "data": "personal.pager.zone",
                "config": {
                    "pagination": {
                        "showSizeChanger": false,
                        "showQuickJumper": true
                    },
                    "size": "large",
                    "action": {
                        "text": "编辑圈子",
                        "uri": "/dp/"
                    },
                    "columns": [
                        {
                            "metadata": "cis,配置项数量,EXPRESSION",
                            "$expr": ":value个",
                            "style": {
                                "fontWeight": 700,
                                "color": "black"
                            }
                        },
                        {
                            "metadata": "added,新增数量,EXPRESSION",
                            "$expr": ":value个",
                            "style": {
                                "fontWeight": 700,
                                "color": "red"
                            }
                        },
                        {
                            "metadata": "createdAt,创建时间,DATE",
                            "$format": "YYYY-MM-DD HH:mm:ss"
                        }
                    ],
                    "mapping": {
                    }
                }
            }
        }
    }
}
```



