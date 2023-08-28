---
title: 如何在webpack中同时使用element v1和v2
date: 2023-08-24
---

# 背景介绍
* "webpack": "^1.13.2", "element-ui": "^1.4.11",
* n年前的项目
* 超过200个路由无法升级


# 代码实现
* compatibleElementV2.js
```js
module.exports = function (source) {
  return source.replace(/element-ui\//g, "element-v2/");
};
```

* webpack.config.js
```js
module.exports = {
  ...
  module: {
    loaders: [

      {
        test: /\.(js|vue)(\?.*$|$)/,
        include: /node_modules\/element-v2/,
        loader: path.resolve(__dirname, "./compatibleElementV2.js")
      },
    ]
  },
  ...
  resolve: {
    alias: {
      "element-v2": path.resolve(__dirname, './node_modules/element-v2'),
    }
  }
  ...
};
```

* package.json
```json
{
  ...
  "dependencies": {
    ...
    "element-v2": "npm:element-ui@2.15",
  }
  ...
}
```

* 测试代码
```vue
<template>
  <div>
    <el-table
      :data="tableData"
      :span-method="arraySpanMethod"
      border
      style="width: 100%">
      <el-table-column
        prop="id"
        label="ID"
        width="180">
      </el-table-column>
      <el-table-column
        prop="name"
        label="姓名">
      </el-table-column>
      <el-table-column
        prop="amount1"
        sortable
        label="数值 1">
      </el-table-column>
      <el-table-column
        prop="amount2"
        sortable
        label="数值 2">
      </el-table-column>
      <el-table-column
        prop="amount3"
        sortable
        label="数值 3">
      </el-table-column>
    </el-table>

    <el-table
      :data="tableData"
      :span-method="objectSpanMethod"
      border
      style="width: 100%; margin-top: 20px">
      <el-table-column
        prop="id"
        label="ID"
        width="180">
      </el-table-column>
      <el-table-column
        prop="name"
        label="姓名">
      </el-table-column>
      <el-table-column
        prop="amount1"
        label="数值 1（元）">
      </el-table-column>
      <el-table-column
        prop="amount2"
        label="数值 2（元）">
      </el-table-column>
      <el-table-column
        prop="amount3"
        label="数值 3（元）">
      </el-table-column>
    </el-table>
  </div>
</template>

<script>
  export default {
    data() {
      return {
        tableData: [{
          id: '12987122',
          name: '王小虎',
          amount1: '234',
          amount2: '3.2',
          amount3: 10
        }, {
          id: '12987123',
          name: '王小虎',
          amount1: '165',
          amount2: '4.43',
          amount3: 12
        }, {
          id: '12987124',
          name: '王小虎',
          amount1: '324',
          amount2: '1.9',
          amount3: 9
        }, {
          id: '12987125',
          name: '王小虎',
          amount1: '621',
          amount2: '2.2',
          amount3: 17
        }, {
          id: '12987126',
          name: '王小虎',
          amount1: '539',
          amount2: '4.1',
          amount3: 15
        }]
      };
    },
    methods: {
      arraySpanMethod({ row, column, rowIndex, columnIndex }) {
        if (rowIndex % 2 === 0) {
          if (columnIndex === 0) {
            return [1, 2];
          } else if (columnIndex === 1) {
            return [0, 0];
          }
        }
      },

      objectSpanMethod({ row, column, rowIndex, columnIndex }) {
        if (columnIndex === 0) {
          if (rowIndex % 2 === 0) {
            return {
              rowspan: 2,
              colspan: 1
            };
          } else {
            return {
              rowspan: 0,
              colspan: 0
            };
          }
        }
      }
    }
  };
</script>
```

* 测试：
![test-demo](/img/elementv1&2-demo.png)
