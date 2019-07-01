---
layout: wiki
title: iview
categories: iview
description: 最初学习iview中的一些记录。
keywords: iview
---

### iview
> 最初学习iview中的一些记录

#### iview 下拉框表单验证

```html  

        ruleValidate2: {
          tag_name: [{ required: true, message: "标签名不能为空", trigger: "blur"}],
          users: [
          {
            required: true,
            type: 'array',
            message: "请选择授权用户",
            trigger: "change"
          }
        ]
        },

```
