---
title: Page rendering
date: 2018-03-26 20:09:51
tags: MEAN
categories: MEAN
---

# 前台页面渲染

## 目录结构

> + public
>    + js
>       + views
>          + bonusType
>             + list
>                + ListItemView.js
>                + ListView.js
>             + CreateView.js
>             + TopBarView.js
>       + router.js
>    + templates
>       + bonusType
>          + list
>             + listHeader.html
>             + listTemplate.html
>          + cancelEdit.html
>          + createTemplate.html
>          + topBarTemplate.html
<!-- more -->

## 分析图

**画的比较烂**╮(~﹏~)╭

![pending Rendering](/lct.png)

## 调用顺序

### index.html

> 加载/js/main.js
>
> 加载/js/app.js
>
> 加载/js/router.js（这里加载了很多重要的文件)



### router.js



#### 得到路由

> 'easyErp/:contentType/list(/pId=:parrentContentId)(/p=:page)(/c=:countPerPage)(/filter=:filter)': 'goToList'



#### 调用方法goToList

> goToList: function (contentType, parrentContentId, page, countPerPage, filter)



#### 调用方法goList

> function goList(context)
>
>  + contentViewUrl = 'views/' + contentType + '/list/ListView';
>  + topBarViewUrl = 'views/' + contentType + '/TopBarView';



#### 解析TopBarView

**(view/:contentType/TopBarView)**

```
var TopBarView = BaseView.extend({
    el         : '#top-bar',
    contentType: CONSTANTS.WORKPOINT,
    template: _.template(ContentTopBarTemplate)
});
```
渲染页面`templates/workPoint/TopBarTemplate.html`(这里是以workPoint为例)



#### 解析ListView

**（view/:contentType/list/ListView）**

渲染页面`templates/workPoint/list/ListHeader.html`

加载文件`views/workPoint/list/ListItemView,views/workPoint/CreateView,views/workPoint/EditView`

(这里是以workPoint为例)