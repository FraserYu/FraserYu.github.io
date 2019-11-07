---
title: Java实现文件下载并压缩成zip文件
tags:
  - IO
categories: [Coding, Java]
id: java-download-file-zip
date: 2018-07-04 13:48:44
description: Java实现文件的下载以及压缩
keywords: Java文件下载,压缩,zip,下载
---

<blockquote class="blockquote-center">IO流，网络编程的基础</blockquote>

### 背景
最近在做Springboot分布式微服务项目，用到了文件上传和下载，文件上传是既有的功能，将文件或图片上传到S3服务器上，将生成的文件URL地址和截取的文件名存储到DB中，正常我们只需要给前端URL，通过浏览器就可以实现文件的下载了，但是我们在将文件上传到S3服务器之前是将文件名称进行转码的，如：文件test.docx上传到S3服务器之后的URL地址是： *http://www.S3.xxx.com/files/abRsdf.docx*, 这样我们下载下来的文件文件名就是 *abRsdf.docx*, 显然这是不符合我们要求的，下载的文件名应该是test.docx, 并且需要多个文件压缩成zip文件打包下载. 所以需要在response中做一些文章，修改文件名，以及做多文件的压缩。

### 实现
因为DB中存储了文件的URL和文件名称，大概数据形式如下：

|序号|url|name|
|:----|:-----|:----|
|1|http://www.S3.xxx.com/files/abRsdf.docx| test.docx|
|2|http://www.S3.xxx.com/files/lsdTew.docx| test1.docx|

<!-- more -->


#### 封装DB数据
将DB中的url和name数据封装到类型为DownloadVO.java中，DownloadVO.java的代码很简单：

```Java
import com.google.common.base.Strings;
import com.viewhigh.epro.mall.vo.base.DownloadVO;
import io.jsonwebtoken.lang.Collections;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.*;
import java.net.HttpURLConnection;
import java.net.MalformedURLException;
import java.net.URL;
import java.util.ArrayList;
import java.util.List;
import java.util.UUID;
import java.util.zip.ZipEntry;
import java.util.zip.ZipOutputStream;

public class DownloadUtil {

    private final static Logger LOG = LoggerFactory.getLogger(DownloadUtil.class);


    /**
     * 下载单个文件
     * @param downloadVO
     * @param request
     * @param response
     * @throws IOException
     */
    public static void toDownloadFile(DownloadVO downloadVO, HttpServletRequest request, HttpServletResponse response)throws IOException {
        String savePath = request.getServletContext().getRealPath(new StringBuffer(File.separator).append("attachment").append(File.separator).toString());
        LOG.info("The Path where file will storage is {}", savePath);

        downloadFile(getFile(downloadVO, savePath), request, response, true);
    }

    /**
     * 在服务器生成文件
     * @param downloadVO
     * @param savePath
     * @return
     * @throws IOException
     */
    public static File getFile(DownloadVO downloadVO, String savePath) throws IOException{
        InputStream inputStream = null;
        FileOutputStream fileOutputStream = null;
        try {
            URL url = new URL(downloadVO.getUrl());

            HttpURLConnection conn = (HttpURLConnection)url.openConnection(); // 创建连接实例
            conn.setConnectTimeout(3*1000); //设置超时间为3秒
            conn.setRequestProperty("User-Agent", "Mozilla/4.0 (compatible; MSIE 5.0; Windows NT; DigExt)"); //防止屏蔽程序抓取而返回403错误
            conn.setRequestProperty("Charset", "UTF-8");


            inputStream = conn.getInputStream(); //获取连接输入流
            byte[] getData = readInputStream(inputStream); //获取字节数组

            File saveDir = new File(savePath); // 创建文件存放路径
            if (!saveDir.exists()){
                saveDir.mkdir();
            }


            File file = new File(savePath,  downloadVO.getName()); // 在新建路径下创建对应附件名称的文件
            fileOutputStream = new FileOutputStream(file);
            fileOutputStream.write(getData);

            return file;

        } catch (MalformedURLException e) {
            throw e;
        } catch (IOException e) {
            throw e;
        }finally {
            // 关闭流
            if(fileOutputStream!=null){
                fileOutputStream.close();
            }
            if(inputStream!=null){
                inputStream.close();
            }
        }
    }

    /**
     * 下载多个文件，并压缩成zip
     * @param downloadVOS 待下载的文件信息
     * @param request Servlet请求
     * @param response Servlet相应
     * @param zipName 待生成的zip文件名成
     * @throws IOException
     * @throws ServletException
     */
    public static void toDownloadFiles(List<DownloadVO> downloadVOS, HttpServletRequest request, HttpServletResponse response, String zipName)throws IOException, ServletException {
        if (!Collections.isEmpty(downloadVOS)){
            LOG.info("The total of Attachment is {}" , downloadVOS.size());

            String savePath = request.getServletContext().getRealPath(new StringBuffer(File.separator).append("attachment").append(File.separator).toString());
            LOG.info("The Path where file will storage is {}", savePath);

            List<File> files = new ArrayList<File>(downloadVOS.size());

            /**
             * 处理同名文件，如果存在，则以 a.txt, a(1).txt, a(2).txt
             */
            Map<String, Integer> map = Maps.newHashMap();
            for (DownloadVO downloadVO : downloadVOS){
                String fileName = downloadVO.getName();
                if (map.containsKey(fileName)){
                    map.put(fileName, map.get(fileName) + 1);
                }else {
                    map.put(fileName, 0);
                }
                if (map.get(fileName) != 0){
                    int position = fileName.lastIndexOf(".");
                    downloadVO.setName(new StringBuffer(fileName.substring(0, position)).append("(").append(map.get(fileName)).append(")").append(fileName.substring(position, fileName.length())).toString());
                }
                File file = getFile(downloadVO, savePath);
                files.add(file);
            }

            // 生成zip文件
            File fileZip = generateZipFile(files, savePath, zipName);

            // 下载zip文件
            downloadFile(fileZip, request, response, true);

            // 删除源文件
            deleteSourceFile(files);
        }
    }

    private static void deleteSourceFile(List<File> files) {
        if (!Collections.isEmpty(files)){
            files.forEach(File::delete);
        }
    }

    /**
     * 从输入流中获取字节数组，根据 available 获取一次性读取的字节数量
     * @param inputStream
     * @return
     * @throws IOException
     */
    public static  byte[] readInputStream(InputStream inputStream) throws IOException {
        int count = 0;
        while (count == 0) {
            count = inputStream.available();
        }
        byte[] buffer = new byte[count];
        int len = 0;
        ByteArrayOutputStream bos = new ByteArrayOutputStream();
        while((len = inputStream.read(buffer)) != -1) {
            bos.write(buffer, 0, len);
        }
        LOG.info("current file length is {}", len);
        bos.close();
        return bos.toByteArray();
    }

    /**
     * 生成zip文件
     * @param files
     * @param savePath
     * @param zipName
     * @return
     */
    public static File generateZipFile(List<File> files, String savePath, String zipName) throws IOException, ServletException {
        String zipFileName = zipName;
        if (Strings.isNullOrEmpty(zipFileName)){
            zipFileName = UUID.randomUUID().toString();
        }
        zipFileName = zipFileName + ".zip";
        File fileZip = new File(savePath, zipFileName );
        FileOutputStream fileOutputStream = null; // 文件输出流
        ZipOutputStream zipOutputStream = null; // 压缩流
        try {
            fileOutputStream = new FileOutputStream(fileZip);
            zipOutputStream = new ZipOutputStream(fileOutputStream);
            zipFile(files, zipOutputStream);
        } catch (FileNotFoundException e) {
            throw e;
        }finally {
            if (zipOutputStream != null){
                zipOutputStream.close();
            }
            if (fileOutputStream != null){
                fileOutputStream.close();
            }
        }

        return fileZip;
    }

    /**
     * 压缩文件
     * @param files
     * @param outputStream
     * @throws IOException
     * @throws ServletException
     */
    public static void zipFile(List<File> files, ZipOutputStream outputStream) throws IOException, ServletException {
        try {
            int size = files.size();
            // 压缩列表中的文件
            for (int i = 0; i < size; i++) {
                File file = (File) files.get(i);
                zipFile(file, outputStream);
            }
        } catch (IOException e) {
            throw e;
        }
    }
    public static void zipFile(File inputFile, ZipOutputStream outputstream) throws IOException, ServletException {
        FileInputStream inStream = null;
        BufferedInputStream bInStream = null;
        try {
            if (inputFile.exists()) {
                if (inputFile.isFile()) {
                    inStream = new FileInputStream(inputFile);
                    bInStream = new BufferedInputStream(inStream);
                    ZipEntry entry = new ZipEntry(inputFile.getName());
                    outputstream.putNextEntry(entry);

                    final int MAX_BYTE = 10 * 1024 * 1024; // 最大的流为10M
                    long streamTotal = 0; // 接受流的容量
                    int streamNum = 0; // 流需要分开的数量
                    int leaveByte = 0; // 文件剩下的字符数
                    byte[] inOutbyte; // byte数组接受文件的数据

                    streamTotal = bInStream.available(); // 通过available方法取得流的最大字符数
                    streamNum = (int) Math.floor(streamTotal / MAX_BYTE); // 取得流文件需要分开的数量
                    leaveByte = (int) streamTotal % MAX_BYTE; // 分开文件之后,剩余的数量

                    if (streamNum > 0) {
                        for (int j = 0; j < streamNum; ++j) {
                            inOutbyte = new byte[MAX_BYTE];
                            // 读入流,保存在byte数组
                            bInStream.read(inOutbyte, 0, MAX_BYTE);
                            outputstream.write(inOutbyte, 0, MAX_BYTE); // 写出流
                        }
                    }
                    // 写出剩下的流数据
                    inOutbyte = new byte[leaveByte];
                    bInStream.read(inOutbyte, 0, leaveByte);
                    outputstream.write(inOutbyte);
                    outputstream.closeEntry(); // Closes the current ZIP entry
                    // and positions the stream for
                    // writing the next entry

                }
            } else {
                throw new ServletException("文件不存在！");
            }
        } catch (IOException e) {
            throw e;
        }finally {
            if (bInStream != null){
                bInStream.close(); // 关闭
            }
            if (inStream != null){
                inStream.close();
            }
        }
    }

    /**
     * 下载zip文件
     * @param file
     * @param response
     * @param isDelete
     */
    public static void downloadFile(File file,HttpServletRequest request , HttpServletResponse response,boolean isDelete) throws IOException {
        BufferedInputStream bufferedInputStream = null;
        OutputStream outputStream = null;
        try {
            bufferedInputStream = new BufferedInputStream(new FileInputStream(file.getPath()));
            byte[] buffer = new byte[bufferedInputStream.available()];
            bufferedInputStream.read(buffer);


            response.reset(); // 清空response
            outputStream = new BufferedOutputStream(response.getOutputStream());
            response.setContentType("application/octet-stream");
            LOG.info("fileName:" + file.getName());

            String userAgent = request.getHeader("User-Agent");
            String formFileName = file.getName();

            // 针对IE或者以IE为内核的浏览器：
            if (userAgent.contains("MSIE") || userAgent.contains("Trident") || userAgent.contains("Edge")) {
                formFileName = java.net.URLEncoder.encode(formFileName, "UTF-8");
            } else {
                // 非IE浏览器的处理：
                formFileName = new String(formFileName.getBytes("UTF-8"), "ISO-8859-1");
            }
            response.setHeader("Content-Disposition", "attachment;filename=" + formFileName);
            outputStream.write(buffer);
            outputStream.flush();

            //是否将生成的服务器端文件删除
            if(isDelete){
                file.delete();
            }
        } catch (IOException ex) {
            throw  ex;
        }finally {
            if (bufferedInputStream != null){
                bufferedInputStream.close();
            }
            if (outputStream != null){
                outputStream.close();
            }
        }
    }
}
```

#### servlet路径问题
上面的代码将文件下载到了servlet的realPath中，顺便总结一下各种路径的差别

请求路径的URL http://localhost:8080/electest/system/elecMenuAction_menuHome.do

```Java
System.out.println("contentType------"+request.getContentType());
System.out.println("requestcontextPath------"+request.getContextPath());
System.out.println("servletPath----"+request.getServletPath());
System.out.println("requestRealPath"+request.getRealPath(""));
System.out.println("realPath------"+request.getServletContext().getRealPath(""));
System.out.println("requestURI------"+request.getRequestURI());
System.out.println("requestURL-------"+request.getRequestURL());
System.out.println("contextPath-------"+request.getServletContext().getContextPath());
```

打印结果

```Java
contentType------application/x-www-form-urlencoded
requestcontextPath------/electest
servletPath----/system/elecMenuAction_menuHome.do
requestRealPathD:\tomcate\apache-tomcat-7.0.63-windows-x64\apache-tomcat-7.0.63\webapps\electest
realPath------D:\tomcate\apache-tomcat-7.0.63-windows-x64\apache-tomcat-7.0.63\webapps\electest
requestURI------/electest/system/elecMenuAction_menuHome.do
requestURL-------http://localhost:8080/electest/system/elecMenuAction_menuHome.do
contextPath---------/electest
```

方法和代码的注释写的已经很详细了，具体的实现就不再详述了
