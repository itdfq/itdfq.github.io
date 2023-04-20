---
title: EasyExecl导出两个sheet
top: false
cover: false
toc: true
mathjax: true
date: 2023-04-20 14:33:54
password:
summary: 本文介绍如何使用EasyExcel写出数据,写入到两个sheet
tags:
- 原创
- EasyExcel
categories:
- 工具
---

具体实现代码如下：

```java
public void export(HttpServletResponse response){
    //设置返回excel的格式为xlsx
    response.reset();
    response.setContentType("application/vnd.openmosix-officedocument.spreadsheetml.sheet;charset=utf-8");
    response.setHeader("Content-Disposition", "attachment;filename=" + URLEncoder.encode("shopee_shop_category", "utf-8") + ".xlsx");
    com.alibaba.excel.ExcelWriter excelWriter = null;
    try (OutputStream os = response.getOutputStream()) {
        excelWriter = EasyExcel.write(response.getOutputStream()).build();
        WriteSheet writeSheet = EasyExcel.writerSheet("sheet1").head(ShopCategoryExportDto.class).build();
        WriteSheet writeSheet2 = EasyExcel.writerSheet("sheet2").head(ShopCategoryItemExportDto.class).build();
        excelWriter.write(exportDtoList, writeSheet);
        excelWriter.write(itemExportDtoList, writeSheet2);
        excelWriter.finish();
    } catch (IOException e) {
        throw new BaseRuntimeException("导出失败" + e.getMessage());
    }
}
```

