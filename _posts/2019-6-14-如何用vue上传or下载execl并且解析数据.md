---
layout:     post
title:      如何用vue上传or下载execl并且解析数据
subtitle:   vue前端解析excel
date:       2019-06-14
author:     yiyiwann
header-img: img/post-bg-universe.jpg
catalog: true
tags:
    - Vue
    - 前端
---


## 前言

目前的项目需要用浏览器下载excel模板，并且把数据填入excel中后上传给浏览器，前端通过该excel解析出它的数据并将其作为参数通过api传给后台。
做完这个功能回头看看，虽然很简单，但是由于很多地方不踏实，踩了很多坑，记录下来做个备忘。

## 正文

#导出excel

-前置准备-
导出excel我之前有做过，这次还是按照网上的教程，加入文件Blob.js和Export2Excel.js
链接：https://pan.baidu.com/s/1W84AeHajdSMkI4dSIfOMSg 
提取码：px0s 
ps：在添加后注意根据文件位置更改Export2Excel.js里blob的加载地址

安装依赖包：
npm install -S file-saver  xlsx
npm install -D script-load

-添加配置-
在webpack.base.conf.js之中的resolve中的alias添加配置
alias: {
      'vue$': 'vue/dist/vue.esm.js',
      '@': resolve('src'),
      'vendor': path.resolve(__dirname, '../src/vendor'),//添加此行
 }
 
 -该页面添加代码-
  download () {
      console.log('下载表格')
      require.ensure([], () => {
        const {
          export_json_to_excel
        } = require('vendor/Export2Excel')
        var tHeader = ['Time', '王小明', '李小梅', '林林', '秦小明'] // excel标题
        var filterVal = ['year', 'CH1', 'CH2', 'CH3', 'CH4']
        var list = this.tableData3  // 将数据赋值给list
        var data = this.formatJson(filterVal, list)
        export_json_to_excel(tHeader, data, '身高数据')
      })
    },
formatJson (filterVal, jsonData) {
      return jsonData.map(v => filterVal.map(j => v[j]))
    },
导出成功。若只需要模板，将list赋值为空即可，


#导入excel并且解析

-html页面-
input标签不仅可以代表文本输入框，将type属性更改一下即可输入各种例如颜色（出现调色盘）、时间、图片甚至文件。
为了获取上传的文件，可以通过input标签，如下：

<input id="upload"  type="file" accept=".xls, .xlsx"  @change="uplodData()"/>
accpet代表着只接受后缀为xxx的文件。

-JS-
uplodData: function () {
      var _this = this
      console.log(this.upData)
      console.log(this.ifUpDta)
      let fileselect = document.querySelector('#upload')
        fileselect.addEventListener('change', function (e) {
          console.log('上传表格')
          console.log(e)
          event.preventDefault()
          let file = e.target.files
          if (file.length <= 0) { // 如果没有文件名
            return
          } else if (!/\.(xls|xlsx)$/.test(file[0].name.toLowerCase())) {
            this.$message({
              type: 'info',
              message: '上传格式不正确，请上传xls或者xlsx格式'
            })
            return
          }
          let reader = new FileReader()
          if (typeof FileReader === 'undefined') {
            this.$message({
              type: 'info',
              message: '您的浏览器不支持FileReader接口'
            })
            return
          }
          reader.readAsBinaryString(file[0])
          reader.onload = function (e) {
            console.log(e)
            try { 
              console.log('转换')
              var data = e.target.result
              var workbook = XLSX.read(data, { type: 'binary' })
              var wsname = workbook.SheetNames[0] // 取第一张表
              var ws = XLSX.utils.sheet_to_json(workbook.Sheets[wsname]) // 生成json表格内容
              console.log(ws)
            } 
            catch (e) {
              return false
            }
          }
        }.bind(this))
    },
  此时最基本的上传获取excel数据功能就实现了。
  
  -个性化问题-
  
 1、根据调试结果，发现addEventListener的change方式只对传了与之前操作时上传的文件不一样才会触发。
  第一次直接上传不会触发，因此在mounted阶段，我把这个函数调用了一遍，那么在渲染时候调用一遍这时是无文件，在我第一次上传文件时就会和之前不同
  
 2、用if条件，在浏览器的输入框中没有填上数据时不触发addEventListener的change，但是发现else里的内容发生了，if里的也发生了。
 该问题就是我进入思维死角钻牛角尖了。根据客户需求，不会在传文件后立刻调用api，而是在确定输入框中填入的数据正确、文件上传正确时才会按下按钮调用api
 
 可以设置另一个按钮，在另一个函数判断有无写入参数，并将excel里的数据也传入该函数。然后在这个按钮里调用api.
 
