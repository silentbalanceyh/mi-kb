# 业务层 Identity

　　业务层 Identity 是专用的模型选择器，这个选择器用于在通道中动态选择模型，API和JOB通道中，主要的模型统一标识有两种：

* 直接在`I_SERVICE`中配置，`identifier`字段的值。
* 通过Identity配置模型选择器（动态处理）。