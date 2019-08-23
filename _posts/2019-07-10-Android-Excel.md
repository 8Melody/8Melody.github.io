---
layout: post
title: "Android操作Excel"
subtitle: "本文记录Android中Excel的读写。"
author: "XYH"
header-img: ""
header-bg-css: "linear-gradient(to right, #24b94a, #38ef7d);"
tags:
  - Android
  - Excel
---

> 记录Android导入/导出Excel。

# 前言

最近工作有需求要读取跟写入Excel文件，最终采用了Apach开源的[POI](https://poi.apache.org/)库来处理。

## 读Excel

以sd卡上的一个POT-Test.xlsx文件为例，权限问题自行处理。

![image](https://qfxl.oss-cn-shanghai.aliyuncs.com/images/poi-excel.png)
```java
  "${Environment.getExternalStorageDirectory()}/POI-Test.xlsx".also { path ->
                File(path).also { f ->
                    if (f.exists()) {
                        try {
                            //获取工作簿
                            XSSFWorkbook(FileInputStream(f)).apply {
                                val evaluator = creationHelper.createFormulaEvaluator()
                                //获取第一个sheet
                                getSheetAt(0).apply {
                                    //当前sheet的所有内容
                                    val content = StringBuilder()
                                    //获取行数
                                    val rowCount = physicalNumberOfRows
                                    //获取每一行的列数
                                    var columnCount: Int
                                    for (r in 0 until rowCount) {
                                        content.append("\n第$r 行数据  ")
                                        columnCount = getRow(r).physicalNumberOfCells
                                        val currentRow = getRow(r)
                                        for (c in 0 until columnCount) {
                                            content.append(getCellString(currentRow, c, evaluator))
                                                .append("  ")
                                        }
                                    }
                                    "数据读取完毕\n $content".log()
                                }
                            }
                        } catch (e: Exception) {
                            e.message?.log()
                        }
                    }
                }
            }
            
private fun getCellString(r: Row, position: Int, formulaEvaluator: FormulaEvaluator): String {
        val cell = r.getCell(position)
        val cellValue = formulaEvaluator.evaluate(cell)
        return when (cellValue.cellType) {
            Cell.CELL_TYPE_NUMERIC -> {
                if (HSSFDateUtil.isCellDateFormatted(cell)) {
                    val date = cellValue.numberValue
                    val formatter = SimpleDateFormat("MM/dd/yy", Locale.CHINA)
                    formatter.format(HSSFDateUtil.getJavaDate(date))
                } else {
                    cellValue.numberValue.toString()
                }
            }
            Cell.CELL_TYPE_STRING -> {
                return cellValue.stringValue
            }
            Cell.CELL_TYPE_BOOLEAN -> {
                return cellValue.booleanValue.toString()
            }
            Cell.CELL_TYPE_BLANK -> {
                return "空"
            }
            else -> {
                cellValue.toString()
            }
        }
    }

```

读取结果如下：

```java
    第0 行数据  姓名  学号  成绩  
    第1 行数据  张三  A101  85.5  
    第2 行数据  李四  B102  60.0  
    第3 行数据  王五  C103  78.0  
    第4 行数据  周六  D104  98.0  
```


## 写Excel

```java
 val dataList = mutableListOf(
                ExcelVo("张三", "A101", 85.5),
                ExcelVo("李四", "B102", 60.0),
                ExcelVo("王五", "C103", 78.0),
                ExcelVo("周六", "D104", 98.0)
            )
            val outputFile = File("${Environment.getExternalStorageDirectory()}/POI-Test-Write.xlsx")
            XSSFWorkbook().also { book ->
                try {
                    //创建sheet
                    book.createSheet("sheet0").apply {
                        //创建行
                        createRow(0).apply {
                            //创建列
                            createCell(0).setCellValue("姓名")
                            createCell(1).setCellValue("学号")
                            createCell(2).setCellValue("成绩")
                        }
                        for (c in 0 until dataList.size) {
                            createRow(c + 1).apply {
                                val vo = dataList[c]
                                createCell(0).setCellValue(vo.name)
                                createCell(1).setCellValue(vo.sn)
                                createCell(2).setCellValue(vo.results)
                            }
                        }
                    }
                    book.write(FileOutputStream(outputFile))
                } catch (e: Exception) {
                    e.message?.log()
                }
            }
```
以上是简单的读写，下面是自定义单元格相关。


## 自定义单元格格式

### 设置行高

```java
val sheet = book.createSheet("sheet0")
val row = sheet.createRow(0)
row.height = 1500
```

### 设置列宽

```java
val sheet = book.createSheet("sheet0")
//设置第一列宽度
sheet.setColumnWidth(0, 5000)
```

### 设置单元格内容对其方式

以水平垂直居中为例

```java
val sheet = book.createSheet("sheet0")
sheet.setColumnWidth(0, 5000)
val row = sheet.createRow(0)
val cell = row.createCell(0)
val cellStyle = book.createCellStyle()
cellStyle.alignment = XSSFCellStyle.ALIGN_CENTER
cellStyle.verticalAlignment = XSSFCellStyle.VERTICAL_CENTER
cell.cellStyle = cellStyle
```

### 设置单元格外边框

```java
val sheet = book.createSheet("sheet0")
sheet.setColumnWidth(0, 5000)
val row = sheet.createRow(0)
val cell = row.createCell(0)
val cellStyle = book.createCellStyle()
cellStyle.borderBottom = XSSFCellStyle.BORDER_DOUBLE
cellStyle.borderLeft = XSSFCellStyle.BORDER_THICK
cellStyle.borderTop = XSSFCellStyle.BIG_SPOTS
cellStyle.topBorderColor = IndexedColors.BLUE.index
cell.cellStyle = cellStyle
```

### 设置内容旋转

```java
val sheet = book.createSheet("sheet0")
sheet.setColumnWidth(0, 5000)
val row = sheet.createRow(0)
val cell = row.createCell(0)
val cellStyle = book.createCellStyle()
cellStyle.rotation = 90
cell.cellStyle = cellStyle
```

### 设置填充色

```java
val sheet = book.createSheet("sheet0")
sheet.setColumnWidth(0, 5000)
val row = sheet.createRow(0)
val cell = row.createCell(0)
val cellStyle = book.createCellStyle()
cellStyle.fillBackgroundColor = HSSFColor.RED.index
cellStyle.fillPattern = XSSFCellStyle.LESS_DOTS
cell.cellStyle = cellStyle
```


完整代码：

```java
val outputFile = File("${Environment.getExternalStorageDirectory()}/POI-1.xlsx")
            XSSFWorkbook().also { book ->
                try {
                    //创建sheet
                    book.createSheet("sheet0").apply {
                        //创建行
                        createRow(0).apply {
                            height = 1500
                            //第二列列宽
                            setColumnWidth(1, 5000)
                            //创建列
                            createCell(0)
                                .setCellValue("测试高度")
                            createCell(1).apply {
                                cellStyle = book.createCellStyle().also { style ->
                                    style.alignment = XSSFCellStyle.ALIGN_CENTER
                                    style.verticalAlignment = XSSFCellStyle.VERTICAL_CENTER
                                }
                            }.setCellValue("测试对其方式")
                            createCell(2).apply {
                                cellStyle = book.createCellStyle().also { style ->
                                    style.borderBottom = XSSFCellStyle.BORDER_DOUBLE
                                    style.borderLeft = XSSFCellStyle.BORDER_THICK
                                    style.borderTop = XSSFCellStyle.BIG_SPOTS
                                    style.topBorderColor = IndexedColors.BLUE.index
                                }
                            }.setCellValue("测试外边框")
                            createCell(3).apply {
                                cellStyle = book.createCellStyle().also { style ->
                                    style.fillBackgroundColor = HSSFColor.RED.index
                                    style.fillPattern = XSSFCellStyle.LESS_DOTS
                                }
                            }.setCellValue("测试填充色")
                            createCell(4).apply {
                                cellStyle = book.createCellStyle().also { style ->
                                    style.rotation = 90
                                }
                            }.setCellValue("测试旋转")
                        }

                    }
                    book.write(FileOutputStream(outputFile))
                } catch (e: Exception) {
                    e.message?.log()
                }
            }
```
结果如图：
![image](https://qfxl.oss-cn-shanghai.aliyuncs.com/images/poi-style.png)

[完整代码。](https://github.com/qfxl/AndroidDemos/tree/master/poi)