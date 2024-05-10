---
title: Vue入门
tags:
  - Vue
categories:
  - Vue
cover: /image/top_img.jpg
top_img: /image/top_img.jpg
abbrlink: df7b5028
date: 2024-04-19 16:28:48
---

# Vue入门

### 1、Vue请求后端

​	`vue`文件，访问后端获取数据不止这一种方法，此方法应该为封装后方法，有其他需要请直接查看官方文档('')，并展示到前端的步骤：

​	1、在`<template>`中完成大概样式，查看参数找到绑定数据的参数用来绑定数据。

```javascript
:data="subList"
```

​	2、上面绑定的数据，需要返回，在`return`中返回`subList`，例如：

```javascript
return {
	subList: generateData(),
}
```

​	3、`generateData()`函数需要在`data()`中定义：

```javascript
data() {
    const generateData = _ => {
    const subList = [];
    test = getSubList();
    subList.push({
        key: 1,
        label: `test`,
        disabled: false,
    })
        return subList;
    };
```

​	4、前端方法实例：

```javascript
getSubList() {
      utils.requiredData(
        "/authPlatform/sysInfConfig/read/list", { 'status': '1' }, (data) => {
          this.subList = [];
          if (data) {
            data.rows.forEach((dt) => {
                this.subList.push({'key': dt.id, 'label': dt.name })
            });
          }
        },
        (err) => {
          utils.showError(err);
        }
      );
    },
```

​	`getSubList`方法中，使用了`requiredData`方法，一般使用三个参数，`requiredData("后端接口路径"， { '属性'： '值' })， (data) => {方法}`，当`{ '属性'： '值' }`为`true`时，数据才可以进入，否则无法进入。

​	下面是中间部分，也是真正取数据的部分：

```javascript
if (data) {
	data.rows.forEach((dt) => {
		this.subList.push({'key': dt.id, 'label': dt.name })
	});
}
```

​		首先`if (data)`判断了数据是否为空，数据中的`rows`是该列表中的一个`key`，遍历`rows`中的每一个元素，然后`push`进`subList`，`subList`为一个对象字面量结构为：`{key: 1, value: 2}`，在`java`中被称为`Map`类型。

​		此时我们就已经拿到了需要的数据并将其封装到了`subList`中（本人小白，不知道这种说法是否准确），我们将其放到对应的组件中使用就可以了。

### 2、Vue绑定字典步骤

​	`MySql`数据库中的`tinyint`对应的是后端的`Boolean`类型，但是为了处理数据，后端一般使用`Integer`类型来返回给前端，前端拿到`Boolean`类型的数据后，在`<script>`标签中，添加一个字典类型`dic`来处理`Boolean`类型的数据，例如：

```java
dicts: ['sys_client_config_type', 'sys_client_config_status'],
```

​	在`<template>`标签中的正确位置上，例如添加下面代码：

```vue
<el-table-column 
	prop="type"
	label="客户端类型, 1: 业务授权"
	width="150">
	<template slot-scope="scope">
		<dict-tag :options="dict.type.sys_client_config_type" :value="scope.row.type"/>
	</template>
</el-table-column>
```

​	注意上面代码中的`<dict-tag>`标签，`:options`中的值，应该对应`dict`中的值，例如`dict`中的值为`sys_client_config_type`，则`:options`中的值应该为`dict.type.sys_client_config_type`，同理`:value`中的值应该对应`prop`中的值。
