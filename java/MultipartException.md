---
title: "解决SpringBoot上传文件异常"
date: 2020-03-24
keywords: 上传文件
description: ""
tags: SpringBoot
author: pugongying
---

#### SpringBoot项目上传文件后报错

>异常信息：

```
exception is org.springframework.web.multipart.MultipartException: Failed to parse multipart servlet request; nested exception is org.apache.commons.fileupload.FileUploadBase$IOFileUploadException: Processing of multipart/form-data request failed. /tmp/tomcat.6749631969221816745.3131/work/Tomcat/localhost/dts/upload_3ba7ace3_a73f_463d_b13b_01fec9b0ae9a_00000522.tmp (No such file or directory)] with root cause
java.io.FileNotFoundException: /tmp/tomcat.6749631969221816745.3131/work/Tomcat/localhost/dts/upload_3ba7ace3_a73f_463d_b13b_01fec9b0ae9a_00000522.tmp (No such file or directory)
        at java.io.FileOutputStream.open0(Native Method)

```

#### 产生原因

找不到Tomcat的临时文件

* 为什么上传的文件要缓存到本地？方便流读取
* 为什么临时目录会不存在？SpringBoot启动时会生成一个临时文件，但是系统会定期删除临时文件


org.apache.tomcat.util.http.fileupload.FileUploadBase#parseRequest，抛出异常的类，代码如下：

```
public List<FileItem> parseRequest(RequestContext ctx)
        throws FileUploadException {
    List<FileItem> items = new ArrayList<>();
    boolean successful = false;
    try {
        FileItemIterator iter = getItemIterator(ctx);
        // 文件工厂类，里面保存了临时目录的地址
        FileItemFactory fac = getFileItemFactory();
        if (fac == null) {
            throw new NullPointerException("No FileItemFactory has been set.");
        }
        while (iter.hasNext()) {
            final FileItemStream item = iter.next();
            // Don't use getName() here to prevent an InvalidFileNameException.
            final String fileName = ((FileItemIteratorImpl.FileItemStreamImpl) item).name;
            // 创建了以上报错信息中的临时文件
            FileItem fileItem = fac.createItem(item.getFieldName(), item.getContentType(),
                                               item.isFormField(), fileName);
            items.add(fileItem);
            try {
                // 流的拷贝，将输入流数据写入输出流
                Streams.copy(item.openStream(), fileItem.getOutputStream(), true);
            } catch (FileUploadIOException e) {
                throw (FileUploadException) e.getCause();
            } catch (IOException e) {
                throw new IOFileUploadException(String.format("Processing of %s request failed. %s",
                                                       MULTIPART_FORM_DATA, e.getMessage()), e);
            }
            final FileItemHeaders fih = item.getHeaders();
            fileItem.setHeaders(fih);
        }
        successful = true;
        return items;
    } catch (FileUploadIOException e) {
        throw (FileUploadException) e.getCause();
    } catch (IOException e) {
        throw new FileUploadException(e.getMessage(), e);
    } finally {
        if (!successful) {
            for (FileItem fileItem : items) {
                try {
                    fileItem.delete();
                } catch (Exception ignored) {
                    // ignored TODO perhaps add to tracker delete failure list somehow?
                }
            }
        }
    }
}

```

以下为创建临时文件

```
/**
 * Creates and returns a {@link java.io.File File} representing a uniquely
 * named temporary file in the configured repository path. The lifetime of
 * the file is tied to the lifetime of the <code>FileItem</code> instance;
 * the file will be deleted when the instance is garbage collected.
 * 文件将被删除在垃圾回收时
 *
 * @return The {@link java.io.File File} to be used for temporary storage.
 */
protected File getTempFile() {
    if (tempFile == null) {
        File tempDir = repository;
        if (tempDir == null) {
            //如果没定义则用系统默认的
            tempDir = new File(System.getProperty("java.io.tmpdir"));
        }

        String tempFileName = format("upload_%s_%s.tmp", UID, getUniqueId());

        tempFile = new File(tempDir, tempFileName);
    }
    return tempFile;
}
```

其中用到的生成随机文件名字的两个方法

```
private static final String UID =
        UUID.randomUUID().toString().replace('-', '_');


private static String getUniqueId() {
    final int limit = 100000000;
    int current = COUNTER.getAndIncrement();
    String id = Integer.toString(current);

    // If you manage to get more than 100 million of ids, you'll
    // start getting ids longer than 8 characters.
    if (current < limit) {
        id = ("00000000" + id).substring(id.length());
    }
    return id;
}
```



首先看下FileItemFactory的实例化位置，在org.apache.catalina.connector.Request#parseParts中，代码如下

```
// Create a new file upload handler
DiskFileItemFactory factory = new DiskFileItemFactory();
try {
    factory.setRepository(location.getCanonicalFile());
} catch (IOException ioe) {
    parameters.setParseFailedReason(FailReason.IO_ERROR);
    partsParseException = ioe;
    return;
}


//其中location代码
// TEMPDIR = "javax.servlet.context.tempdir";
location = ((File) context.getServletContext().getAttribute(ServletContext.TEMPDIR));
```


#### 解决问题

* 方法1

应用重启

* 方法2

增加服务配置，自定义baseDir

```
server.tomcat.basedir=/tmp/tomcat
```

* 方法3

注入bean，手动配置临时目录

```
@Bean
MultipartConfigElement multipartConfigElement() {
    MultipartConfigFactory factory = new MultipartConfigFactory();
    factory.setLocation("/tmp/tomcat");
    return factory.createMultipartConfig();
}

```
 
* 方法4

配置不删除tmp目录下的tomcat

```
vim /usr/lib/tmpfiles.d/tmp.conf

# 添加一行
x /tmp/tomcat.*
```


> 微信公众号：码上烟火