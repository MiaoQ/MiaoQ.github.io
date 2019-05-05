---
title: 导出json数据到excel
date: 2019-05-05 16:07:21
tags: tips
gitment: 导出json数据到excel
description: 本文简述了前端如何使用工具导出json数据到excel中
---

### 安装依赖项 ###
```jsx
npm install file-saver --save
npm install xlsx --save
npm install script-loader --save-dev
```

### 下载文件 ###
下载所需文件[Blob.js和Export2Excel.js](http://xiazai.jb51.net/201708/yuanma/Export2Exce_jb51.rar)

### 代码组织部分 ###
1. 在src目录下新建excel文件夹，将上面下载的Blob.js和Export2Excel.js放入excel文件夹中
2. 在存放model的文件中（我这里是store目录下的common.js）添加如下函数，实现将json数据导出为excel文件：
```jsx
export2Excel(context, param) { // 导出excel
  require.ensure([], () => {
    const { export_json_to_excel } = require('../excel/Export2Excel')
    const { fileName, header, headerKey, dataSource } = param
    const data = dataSource.map(v => headerKey.map(j => v[j]))
    export_json_to_excel(header, data, fileName)
  })
}
```
3. 当向后台发起“导出”的ajax请求并得到响应后，返回的json数据大致如下：
```json
res.data = [{
  "id": "01",
  "name": "张三",
  "age": 16,
  "location": "上海"
}, {
  "id": "02",
  "name": "李四",
  "age": 19,
  "location": "杭州"
}]
```
在成功得到返回数据后，调用export2Excel，将数据导出：
```jsx
export2Excel({
  fileName: '人员信息记录表',
  header: ['编号', '姓名', '年龄', '地址'],
  headerKey: ['id', 'name', 'age', 'location'],
  dataSource: res.data
})
```


