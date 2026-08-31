+++
title = "Struts2 下载文件异常解决"
date = 2018-03-19T22:16:00+08:00
lastmod = 2022-04-10T01:47:50+08:00
tags = ["Java", "Struts2"]
categories = ["Java"]
draft = false
toc = true
+++

## 异常描述 {#异常描述}


### 异常截图 {#异常截图}

{{< figure src="https://waver.me/album/images/post/2018/2018_03_19_Srtuts2_Exception.png" >}}


### Java 代码如下 {#java 代码如下}

```java
package action.upload;

import com.opensymphony.xwork2.ActionContext;
import com.opensymphony.xwork2.ActionSupport;

import java.io.File;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.InputStream;
import java.util.Map;

/**
 * @author : Waver
 * @date : 2018/3/19 20:35
 */
public class DownloadAction extends ActionSupport {

    /**
     * 获取文件路径
     */
    private String savePath;
    /**
     * 接收的文件名
     */
    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getSavePath() {
        return savePath;
    }

    public void setSavePath(String savePath) {
        this.savePath = savePath;
    }

    public String list() {
        File file = new File(savePath);
        String[] list = file.list();
        ActionContext context = ActionContext.getContext();
        Map<String, Object> contextMap = context.getContextMap();
        contextMap.put("list", list);
        return "list";
    }

    public InputStream getInputStream() {
        try {
            FileInputStream fileInputStream = new FileInputStream(new File(savePath + name));
            return fileInputStream;
        } catch (FileNotFoundException e) {
            e.printStackTrace();
            throw new RuntimeException("下载异常!!!");
        }
    }

    public String down(){
        return "down";
    }


}
```


### xml 配置信息 {#xml 配置信息}

```text
<struts>
    <package name="upload" extends="struts-default" namespace="/upload">
        <action name="upload" class="action.upload.UploadAction" method="execute">
            <interceptor-ref name="defaultStack">
                <param name="fileUpload.allowedTypes">image/jpeg,image/jpg</param>
            </interceptor-ref>
            <param name="savePath">e:/upload/</param>
            <result>/login.jsp</result>
            <result name="input">/error.jsp</result>
        </action>

        <action name="down_*" class="action.upload.DownloadAction" method="{1}">
            <param name="savePath">e:/upload</param>
            <result name="list">/listFile.jsp</result>
            <result type="stream" name="down">
                <!--返回给浏览器的文件类型, 返回通常用二进制-->
                <param name="contentType">applicantion/octet-stream</param>
                <!--返回给浏览器的输入流-->
                <param name="inputName">inputStream</param>
                <!--告诉浏览器以下载的方式下载资源-->
                <param name="contentDisposition">attachment=${name}</param>
                <!--缓存大小-->
                <param name="bufferSize">1024</param>
            </result>
        </action>
    </package>
</struts>
```


## 错误原因及解决方案 {#错误原因及解决方案}

`<param name`"savePath"&gt;e:/upload&lt;/param&gt;= 项后没有加 **"/"**
与文件名组合后无法找到文件名应改为
`<param name`"savePath"&gt;e:/upload/&lt;/param&gt;=
