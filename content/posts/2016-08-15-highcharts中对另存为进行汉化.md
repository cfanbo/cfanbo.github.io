---
title: highcharts中对”另存为”进行汉化
author: admin
type: post
date: 2016-08-15T08:55:38+00:00
url: /archives/17087
categories:
 - 前端设计
tags:
 - Highcharts

---

```
$(function () {
    var chart = new Highcharts.Chart({
        chart: {
            renderTo: 'container'
        },
       lang｛
                  printChart:"打印图表",
                  downloadJPEG: "下载JPEG 图片"
                  downloadPDF: "下载PDF文档"
                  downloadPNG: "下载PNG 图片"
                  downloadSVG: "下载SVG 矢量图"
                  exportButtonTitle: "导出图片"
      ｝，
  .....
  })
})

```