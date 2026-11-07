---
type: memo
date: 2026-11-06
aliases:
  - "EasyExcel入门"
---
# EasyExcel入门

## 1 介绍

根据需求，目标是将订单趋势图的中的数据导出Excel，数据来源于按天统计表和按小时统计表，现在关键问题是使用java语言生成Excel文档。

Java解析、生成Excel比较有名的框架有Apache poi、jxl。但他们都存在一个严重的问题就是非常的耗内存，poi有一套SAX模式的API可以一定程度的解决一些内存溢出的问题，但POI还是有一些缺陷，比如07版Excel解压缩以及解压后存储都是在内存中完成的，内存消耗依然很大。

Easyexcel重写了poi对07版Excel的解析，一个3M的excel用POI sax解析依然需要100M左右内存，改用easyexcel可以降低到几M，并且再大的excel也不会出现内存溢出；03版依赖POI的sax模式，在上层做了模型转换的封装，让使用者更加简单方便。

EasyExcel是一个基于Java的、快速、简洁、解决大文件内存溢出的Excel处理工具。他能让你在不用考虑性能、内存的等因素的情况下，快速完成Excel的读、写等功能。

- 官方网站：[https://easyexcel.opensource.alibaba.com/](https://easyexcel.opensource.alibaba.com/)
- github地址：[https://github.com/alibaba/easyexcel](https://github.com/alibaba/easyexcel)
- gitee地址：[https://gitee.com/easyexcel/easyexcel](https://gitee.com/easyexcel/easyexcel)

参考官方的例子

## 2 **读Excel**

DEMO代码地址：[https://github.com/alibaba/easyexcel/blob/master/easyexcel-test/src/test/java/com/alibaba/easyexcel/test/demo/read/ReadTest.java](https://github.com/alibaba/easyexcel/blob/master/easyexcel-test/src/test/java/com/alibaba/easyexcel/test/demo/read/ReadTest.java)

```Java
    
package com.jzo2o.orders.history.easyexcel.read;

import java.io.File;
import java.io.InputStream;
import java.util.List;
import java.util.Map;

import com.alibaba.excel.EasyExcel;
import com.alibaba.excel.ExcelReader;
import com.alibaba.excel.annotation.ExcelProperty;
import com.alibaba.excel.annotation.format.DateTimeFormat;
import com.alibaba.excel.annotation.format.NumberFormat;
import com.alibaba.excel.context.AnalysisContext;
import com.alibaba.excel.converters.DefaultConverterLoader;
import com.alibaba.excel.enums.CellExtraTypeEnum;
import com.alibaba.excel.read.listener.PageReadListener;
import com.alibaba.excel.read.listener.ReadListener;
import com.alibaba.excel.read.metadata.ReadSheet;
import com.alibaba.excel.util.ListUtils;

import com.alibaba.fastjson.JSON;
import com.jzo2o.orders.history.easyexcel.util.TestFileUtil;
import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.Test;

/**
 * 读的常见写法
 *
 * @author Jiaju Zhuang
 */

@Slf4j
public class ReadTest {

    /**
     * 最简单的读
     * <p>1. 创建excel对应的实体对象 参照{@link DemoData}
     * <p>2. 由于默认一行行的读取excel，所以需要创建excel一行一行的回调监听器，参照{@link DemoDataListener}
     * <p>3. 直接读即可
     */
    @Test
    public void simpleRead() {
        String fileName = TestFileUtil.getPath() + "demo" + File.separator + "demo.xlsx";
        // 这里 需要指定读用哪个class去读，然后读取第一个sheet 文件流会自动关闭
        EasyExcel.read(fileName, DemoData.class, new DemoDataListener()).sheet().doRead();
    }
    ...
```

运行上边的测试方法，跟踪DemoDataListener类invoke方法，观察控制台输出了从Excel中读取到的数据。



## 3 **写Excel**

DEMO代码地址：[https://github.com/alibaba/easyexcel/blob/master/easyexcel-test/src/test/java/com/alibaba/easyexcel/test/demo/write/WriteTest.java](https://github.com/alibaba/easyexcel/blob/master/easyexcel-test/src/test/java/com/alibaba/easyexcel/test/demo/write/WriteTest.java)

```Java
    
package com.jzo2o.orders.history.easyexcel.write;

import java.io.File;
import java.io.InputStream;
import java.net.URL;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.Date;
import java.util.HashSet;
import java.util.List;
import java.util.Set;

import com.alibaba.excel.EasyExcel;
import com.alibaba.excel.ExcelWriter;
import com.alibaba.excel.annotation.ExcelProperty;
import com.alibaba.excel.annotation.format.DateTimeFormat;
import com.alibaba.excel.annotation.format.NumberFormat;
import com.alibaba.excel.annotation.write.style.ColumnWidth;
import com.alibaba.excel.annotation.write.style.ContentRowHeight;
import com.alibaba.excel.annotation.write.style.HeadRowHeight;
import com.alibaba.excel.enums.CellDataTypeEnum;
import com.alibaba.excel.metadata.data.CommentData;
import com.alibaba.excel.metadata.data.FormulaData;
import com.alibaba.excel.metadata.data.HyperlinkData;
import com.alibaba.excel.metadata.data.HyperlinkData.HyperlinkType;
import com.alibaba.excel.metadata.data.ImageData;
import com.alibaba.excel.metadata.data.ImageData.ImageType;
import com.alibaba.excel.metadata.data.RichTextStringData;
import com.alibaba.excel.metadata.data.WriteCellData;
import com.alibaba.excel.util.BooleanUtils;
import com.alibaba.excel.util.FileUtils;
import com.alibaba.excel.util.ListUtils;
import com.alibaba.excel.write.handler.CellWriteHandler;
import com.alibaba.excel.write.handler.context.CellWriteHandlerContext;
import com.alibaba.excel.write.merge.LoopMergeStrategy;
import com.alibaba.excel.write.metadata.WriteSheet;
import com.alibaba.excel.write.metadata.WriteTable;
import com.alibaba.excel.write.metadata.style.WriteCellStyle;
import com.alibaba.excel.write.metadata.style.WriteFont;
import com.alibaba.excel.write.style.HorizontalCellStyleStrategy;
import com.alibaba.excel.write.style.column.LongestMatchColumnWidthStyleStrategy;

import com.jzo2o.orders.history.easyexcel.util.TestFileUtil;
import org.apache.poi.ss.usermodel.Cell;
import org.apache.poi.ss.usermodel.CellStyle;
import org.apache.poi.ss.usermodel.FillPatternType;
import org.apache.poi.ss.usermodel.IndexedColors;
import org.apache.poi.ss.usermodel.Workbook;
import org.apache.poi.xssf.streaming.SXSSFSheet;
import org.junit.jupiter.api.Test;

/**
 * 写的常见写法
 *
 * @author Jiaju Zhuang
 */

public class WriteTest {
    /**
     * 最简单的写
     * <p>1. 创建excel对应的实体对象 参照{@link com.alibaba.easyexcel.test.demo.write.DemoData}
     * <p>2. 直接写即可
     */
    @Test
    public void simpleWrite() {
        String fileName = TestFileUtil.getPath() + "write" + System.currentTimeMillis() + ".xlsx";
        // 这里 需要指定写用哪个class去读，然后写到第一个sheet，名字为模板 然后文件流会自动关闭
        // 如果这里想使用03 则 传入excelType参数即可
        EasyExcel.write(fileName, DemoData.class).sheet("模板").doWrite(data());
    }
    ...
```

运行上边的测试方法，跟踪写到磁盘的excel文件（fileName ），查看文件内容是否是data()方法返回的数据。



## 4 **web上传、下载**

DEMO代码地址：[https://github.com/alibaba/easyexcel/blob/master/easyexcel-test/src/test/java/com/alibaba/easyexcel/test/demo/web/WebTest.java](https://github.com/alibaba/easyexcel/blob/master/easyexcel-test/src/test/java/com/alibaba/easyexcel/test/demo/web/WebTest.java)

```Java
   
package com.jzo2o.orders.history.controller.test;

import com.alibaba.excel.EasyExcel;
import com.alibaba.excel.util.ListUtils;
import com.jzo2o.orders.history.model.dto.request.HistoryOrdersListQueryReqDTO;
import com.jzo2o.orders.history.model.dto.response.HistoryOrdersDetailResDTO;
import com.jzo2o.orders.history.model.dto.response.HistoryOrdersListResDTO;
import com.jzo2o.orders.history.service.IHistoryOrdersService;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiImplicitParam;
import io.swagger.annotations.ApiOperation;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import javax.annotation.Resource;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.net.URLEncoder;
import java.util.Date;
import java.util.List;


@Controller
@RequestMapping("/excel/test")
public class ExcelController {

/**
     * 文件下载（失败了会返回一个有部分数据的Excel）
     * <p>
     * 1. 创建excel对应的实体对象 参照{@link DownloadData}
     * <p>
     * 2. 设置返回的 参数
     * <p>
     * 3. 直接写，这里注意，finish的时候会自动关闭OutputStream
     */
    @GetMapping("download")
    public void download(HttpServletResponse response) throws IOException {
        // 这里注意 有同学反应使用swagger 会导致各种问题，请直接用浏览器或者用postman
        response.setContentType("application/vnd.openxmlformats-officedocument.spreadsheetml.sheet");
        response.setCharacterEncoding("utf-8");
        // 这里URLEncoder.encode可以防止中文乱码 当然和easyexcel没有关系
        String fileName = URLEncoder.encode("测试", "UTF-8").replaceAll("\\+", "%20");
        response.setHeader("Content-disposition", "attachment;filename*=utf-8''" + fileName + ".xlsx");
        EasyExcel.write(response.getOutputStream(), DemoData.class).sheet("模板").doWrite(data());
       //设置返回body无需包装标识
       response.setHeader(BODY_PROCESSED, "1");                
    }

    /**
     * 文件上传
     * <p>1. 创建excel对应的实体对象 参照{@link UploadData}
     * <p>2. 由于默认一行行的读取excel，所以需要创建excel一行一行的回调监听器，参照{@link UploadDataListener}
     * <p>3. 直接读即可
     */
    @PostMapping("upload")
    @ResponseBody
    public String upload(MultipartFile file) throws IOException {
        EasyExcel.read(file.getInputStream(), DemoData.class, new DemoDataListener()).sheet().doRead();
        return "success";
    }
```

测试上边的download方法。

---
> 来源：飞书技术知识库（脱敏版） · 07 业务组件方案集 / 云岚到家项目 / 第九章 系统优化 / EasyExcel入门 · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/VIiMwXUAQiAAbnknORHcis3NnBg)
