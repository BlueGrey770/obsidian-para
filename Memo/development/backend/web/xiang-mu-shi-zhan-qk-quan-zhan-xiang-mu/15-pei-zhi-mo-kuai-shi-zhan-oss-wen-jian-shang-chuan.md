---
type: memo
date: 2026-11-06
aliases:
  - "15 配置模块实战（OSS文件上传）"
---
# 15 配置模块实战（OSS文件上传）

## 做一个配置模块

以aliyunOSS文件上传为案例：

目录结构

![这张图片展示的是配置模块实战（OSS文件上传）案例中的项目目录结构，属于该实战步骤里目录结构对应的实际工程层级。该目录从src目录展开，main目录下分为java和resources两个子目录；java目录下按包路径com.itheima划分，包含AliOssConfigure、AliProperties等多个功能相关的类文件。resources目录下存在META-INF/spring子目录，该子目录内有一个名为org.springframework.boot.autoconfigure.AutoConfiguration.imports的配置文件，与文档中配置文件编写的要求相对应。](../../../../../Attachments/development/backend/web/xiang-mu-shi-zhan-qk-quan-zhan-xiang-mu/VssxbQFLhoDl3CxjE24chIXNnld.png)

### 将功能类写好

```Java
/**
 * 利用hash算法创建八级目录
 *  可以存储40亿+的数据
 */
public class HashUtils {

    public static String getSubDir(String originalFilename) {
        //1.得到文件的hash值
        int hash = originalFilename.hashCode();
        //2.算出8级目录
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < 8; i++) {
            int sunDir = hash & 15;
            sb.append(sunDir).append("/");
            hash >>>= 4;
        }
        return sb.toString();

    }
}
```

**在pom.xml文件中引入要使用的依赖**

```XML
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>

    <dependency>
        <groupId>com.aliyun.oss</groupId>
        <artifactId>aliyun-sdk-oss</artifactId>
        <version> 3.17.4</version>
    </dependency>

    <dependency>
        <groupId>javax.xml.bind</groupId>
        <artifactId>jaxb-api</artifactId>
        <version> 2.3.1</version>
    </dependency>
    <dependency>
        <groupId>javax.activation</groupId>
        <artifactId>activation</artifactId>
        <version> 1.1.1</version>
    </dependency>
    <!-- no more than 2.3.3-->
    <dependency>
        <groupId>org.glassfish.jaxb</groupId>
        <artifactId>jaxb-runtime</artifactId>
        <version> 2.3.3</version>
    </dependency>

</dependencies>
```

```Java
@Data
public class OSSUtils {

    @Autowired
    private AliProperties aliProperties;

    public  String upload(File src) throws com.aliyuncs.exceptions.ClientException, FileNotFoundException {
        return upload(new FileInputStream(src), src.getName());
    }

    public  String upload(MultipartFile file) throws com.aliyuncs.exceptions.ClientException, IOException {
        return upload(file.getInputStream(), file.getOriginalFilename());
    }

    public  String upload(InputStream in, String fileName) throws com.aliyuncs.exceptions.ClientException {
        // Endpoint以华东1（杭州）为例，其它Region请按实际情况填写。
//        String endpoint = "https://oss-cn-beijing.aliyuncs.com";
        // 从环境变量中获取访问凭证。运行本代码示例之前，请确保已设置环境变量OSS_ACCESS_KEY_ID和OSS_ACCESS_KEY_SECRET。
        EnvironmentVariableCredentialsProvider credentialsProvider = CredentialsProviderFactory.newEnvironmentVariableCredentialsProvider();
        // 填写Bucket名称，例如examplebucket。
//        String bucketName = "xiaoliuaixuexiya";
        // 填写Object完整路径，完整路径中不能包含Bucket名称，例如exampledir/exampleobject.txt。
        String objectName = HashUtils.getSubDir(fileName) + UUID.randomUUID() + "_" + fileName;
        // 填写Bucket所在地域。以华东1（杭州）为例，Region填写为cn-hangzhou。
//        String region = "cn-beijing";

        // 创建OSSClient实例。
        // 当OSSClient实例不再使用时，调用shutdown方法以释放资源。
        ClientBuilderConfiguration clientBuilderConfiguration = new ClientBuilderConfiguration();
        clientBuilderConfiguration.setSignatureVersion(SignVersion.V4);
        OSS ossClient = OSSClientBuilder.create()
                .endpoint(aliProperties.getEndpoint())
                .credentialsProvider(credentialsProvider)
                .clientConfiguration(clientBuilderConfiguration)
                .region(aliProperties.getRegion())
                .build();
        try {

            // 创建PutObjectRequest对象。
            PutObjectRequest putObjectRequest = new PutObjectRequest(aliProperties.getBucketName(), objectName, in);

            // 如果需要上传时设置存储类型和访问权限，请参考以下示例代码。
            // ObjectMetadata metadata = new ObjectMetadata();
            // metadata.setHeader(OSSHeaders.OSS_STORAGE_CLASS, StorageClass.Standard.toString());
            // metadata.setObjectAcl(CannedAccessControlList.Private);
            // putObjectRequest.setMetadata(metadata);

            // 上传字符串。
            PutObjectResult result = ossClient.putObject(putObjectRequest);

            return "https://" + aliProperties.getBucketName()+"."+ aliProperties.getEndpoint().substring("https://".length()) + "/" + objectName;
        } catch (OSSException oe) {
            System.out.println("Caught an OSSException, which means your request made it to OSS, "
                    + "but was rejected with an error response for some reason.");
            System.out.println("Error Message:" + oe.getErrorMessage());
            System.out.println("Error Code:" + oe.getErrorCode());
            System.out.println("Request ID:" + oe.getRequestId());
            System.out.println("Host ID:" + oe.getHostId());
        } catch (ClientException ce) {
            System.out.println("Caught an ClientException, which means the client encountered "
                    + "a serious internal problem while trying to communicate with OSS, "
                    + "such as not being able to access the network.");
            System.out.println("Error Message:" + ce.getMessage());
        } finally {
            if (ossClient != null) {
                ossClient.shutdown();
            }
        }
        return null;
    }
}
```

```TypeScript
@Data
@ConfigurationProperties(prefix = "ali.oss")
public class AliProperties {
        private String endpoint;     //自己创建的aliyun桶所在服务器的url地址
        private String bucketName;   //自己取的aliyun的存储OSS服务的名字
        private String region;       //服务器所在的位置
        private String accessKey;
        private String accessSecret;
        private String[] hobby;
}
```

```TypeScript
public class AliOssConfigure {
    @Bean
    public OSSUtils getOssUtils(){
        return new OSSUtils();
    }

    @Bean
    public DeleteFile getDeleteFile(){
        return new DeleteFile();
    }

    @Bean
    public AliProperties getAliProperties(){
        return new AliProperties();
    }

}
```

### 写一个配置文件

在resource下面创建一个文件夹 MATE-INF/spring

在spring下面写一个文件，命名：org.springframework.boot.autoconfigure.AutoConfiguration.imports

```Java
com.itheima.AliOssConfigure

这里的内容就是前面写的配置类的全限定类名
```

## 做一个起步依赖模块

> 要按照命名规范 xxx-spring-boot-starter

<callout emoji="📌">
这个模块中就要一个pom.xml文件就行其他的都可以删除  
这里面就需要引入一个配置模块的依赖
</callout>

```XML
    <dependencies>
        <dependency>
            <groupId>cn.itcast</groupId>
            <artifactId>alioss-spring-boot-autoconfigure</artifactId>
            <version>0.0.1-SNAPSHOT</version>
        </dependency>
    </dependencies>
```

## 使用起步依赖

<callout emoji="❤️">
如果想要在其他的模块中使用到aliyunOSS的的文件上传功能，只需要引入起步依赖的GAV就可以在业务中通过@Autowired注入需要的功能直接开箱即用了
</callout>

```Java
<dependency>
    <groupId>cn.itcast</groupId>
    <artifactId>alioss-spring-boot-starter</artifactId>
    <version>0.0.1-SNAPSHOT</version>
</dependency>
```

---
> 来源：飞书技术知识库（脱敏版） · 03 Web 开发与框架 / 项目实战：qk 全栈项目 / 15 配置模块实战（OSS文件上传） · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/PaOMwwGLPiydiZkeQuEckuy3nwc)
