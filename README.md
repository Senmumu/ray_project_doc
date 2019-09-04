
# Chinese (zh-cn) translation of the ray project docs

https://ray.readthedocs.io/en/latest/

Translation is not done yet! 文档未完成，欢迎进一步修订。

翻译过程中，请直接将 `sources/` 中 `.md` 文件中的英文替换为中文。


## 排版规范 Typesetting

此文档遵循 [中文排版指南](https://github.com/sparanoid/chinese-copywriting-guidelines) 规范，并在此之上遵守以下约定：

* 英文的左右保持一个空白，避免中英文字黏在一起；
* 使用全角标点符号；
* 严格遵循 Markdown 语法；
* 原文中的双引号（" "）请代换成中文的引号（「」符号怎么打出来见 [这里](http://zhihu.com/question/19755746/answer/27233392)）；
* 「`加亮`」和「**加粗**」和「[链接]()」都需要在左右保持一个空格。

## 翻译对照列表 Conventions

- 该翻译用于 `zh-cn` （简体中文，中国大陆地区）。
- 当遇到以下 `专业术语` 的时候，请使用以下列表进行对照翻译。（未完待续）


| English            | 中文                 |
|:-------------------|:--------------------|
| arguments          | 参数                 |
| boolean            | 布尔                 |
| data augumentation | 数据增强             |
| deep learning      | 深度学习             |
| float              | 浮点数               |
| Functional API     | 函数式 API           |
| Fuzz factor        | 模糊因子             |
| input shape        | 输入尺寸             |
| index              | 索引                 |
| int                | 整数                 |
| layer              | 层                  |
| loss function      | 损失函数             |
| metrics            | 评估标准             |
| nD tensor          | nD 张量             |
| Numpy Array        | Numpy 矩阵            |
| objective          | 目标                 |
| optimizer          | 优化器               |
| output shape       | 输出尺寸             |
| regularizer        | 正则化器             |
| return             | 返回                 |
| recurrent          | 循环                 |
| Sequential Model   | 顺序模型              |
| shape              | 尺寸                 |
| target             | 目标                 |
| testing            | 测试                 |
| training           | 训练                 |
| wrapper            | 封装器               |


Welcome to contribute!!!

# Ray Documentation

编译文档请运行下列的命令。请注意Ray必须先装好。

```
pip install -r requirements-doc.txt
make html
open _build/html/index.html
```

如下操作，测试是否有错误。
```
sphinx-build -W -b html -d _build/doctrees source _build/html
```