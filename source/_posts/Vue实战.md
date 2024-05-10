---
title: Vue实战
tags:
  - Vue
categories:
  - Vue
cover: /image/top_img.jpg
top_img: /image/top_img.jpg
abbrlink: ae9f4892
date: 2024-05-07 13:58:29
---

​	在`dialog`中添加一个`form`时，设置该`form`可编辑，写法如下：

```vue
<el-table :data="gatewayProtocolItem" :rules="rules" class="screen-conent"
          header-row-class-name="custom-table-header" table-class="custom-table">
	<el-table-column fixed type="selection" width="55">
	</el-table-column>
	<el-table-column label="属性名称" prop="propName">
		<template slot-scope="scope">
			<el-input v-if="scope.row.edit" v-model="scope.row.propName"
					 :required="scope.row.required"></el-input>
			<span v-else>{{ scope.row.propName }}</span>
		</template>
	</el-table-column>
	<el-table-column label="功能码" prop="funCode">
		<template slot-scope="scope">
			<el-input v-if="scope.row.edit" v-model="scope.row.funCode"></el-input>
			<span v-else>{{ scope.row.funCode }}</span>
		</template>
	</el-table-column>
</el-table>
```

```js
// 在<el-form>中添加一个新的空行
formItemAdd() {
            const newLine = {}
            this.gatewayProtocolItem.push(newLine)
            let lastIndex = this.gatewayProtocolItem.length - 1
            this.handleEdit(this.gatewayProtocolItem[lastIndex])
        },
```

	当无法使用`required`来控制`input`输入框是否可以为空时，可以使用下面的方法来判断：dialog`

```js
// 遍历表格数据，检查所有必填字段是否为空
handleSubmit(formName) {
	for (let i = 0; i < this.gatewayProtocolItem.length; i++) {
	let row = this.gatewayProtocolItem[i]
	if (!row.propName || !row.funCode ) {
	this.$message.error('请填写完整信息')
	return // 如果有字段为空，则直接返回，不继续提交表单
		}
	}
	// 如果所有必填字段都不为空，则继续提交表单
	this.submitForm(formName)
},
```

​	持续更新...
