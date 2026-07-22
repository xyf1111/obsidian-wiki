---
title: "Docker - MinIO 对象存储服务搭建"
date: 2026-07-22
tags: [Docker, MinIO, 对象存储, DevOps, SpringBoot]
source: "鱼皮·编程导航 / codefather"
---

# Docker - MinIO 对象存储服务搭建

> 使用 Docker 在服务器上搭建 MinIO 高性能对象存储服务，并通过 SpringBoot 集成实现文件上传功能。

## 为什么选择 MinIO

[MinIO](https://min.io) 是一个高性能、轻量级的对象存储服务器，兼容 Amazon S3 API。GitHub 上拥有 43k+ Star，阿里巴巴、腾讯、百度等企业均有使用。

**优势：**
- 低延迟、高吞吐量
- 易于部署和管理（单容器即可运行）
- 分布式、高可用、水平扩展
- 控制成本，避免第三方对象存储被攻击刷流量的风险

## 使用 Docker 搭建 MinIO

### 1. 创建数据目录

```bash
mkdir -p /home/minio/config   # MinIO 配置文件
mkdir -p /home/minio/data     # 上传文件存储
```

### 2. 启动 MinIO 容器

```bash
docker run -p 9000:9000 -p 9001:9001 \
  -d --restart=always \
  -e "MINIO_ACCESS_KEY=admin" \
  -e "MINIO_SECRET_KEY=password" \
  -v /home/minio/data:/data \
  -v /home/minio/config:/root/.minio \
  minio/minio server /data \
  --console-address ":9001"
```

**参数说明：**

| 参数 | 说明 |
|---|---|
| `-p 9000:9000` | API 端口（文件读写） |
| `-p 9001:9001` | Web 控制台端口 |
| `-d --restart=always` | 后台运行，异常自动重启 |
| `MINIO_ACCESS_KEY / MINIO_SECRET_KEY` | 控制台登录账号密码 |
| `-v /home/minio/data:/data` | 持久化文件存储目录 |

### 3. 控制台配置

访问 `http://服务器IP:9001`，使用配置的账号密码登录：

1. **创建 Bucket（存储桶）** — 用于存放上传文件
2. **生成 Access Key / Secret Key** — 用于程序远程访问
3. **添加只读访问规则**（可选）— 若需允许直接通过 URL 访问文件，在 Bucket 的权限配置中添加 `readonly` 规则

### 4. 测试文件访问

上传文件后，访问路径格式：
```
http://服务器IP:9000/{bucket名称}/{文件名}
```

## SpringBoot 集成 MinIO

### 1. 引入依赖

```xml
<!-- MinIO 客户端 -->
<dependency>
    <groupId>io.minio</groupId>
    <artifactId>minio</artifactId>
    <version>8.5.9</version>
</dependency>
```

### 2. 配置类

```java
@Configuration
@ConfigurationProperties(prefix = "file.minio")
@Data
public class MinioConfiguration {

    private String accessKey;
    private String secretKey;
    private String endpoint;
    private String bucket;

    @Bean
    public MinioClient minioClient() {
        return MinioClient.builder()
                .endpoint(endpoint)
                .credentials(accessKey, secretKey)
                .build();
    }
}
```

### 3. application.yml 配置

```yaml
file:
  minio:
    endpoint: http://服务器ip:9000
    accessKey: <生成的 AccessKey>
    secretKey: <生成的 SecretKey>
    bucket: guanzhi
```

### 4. 文件上传接口

```java
@RestController
@RequestMapping("/file")
public class FileController {

    @Resource
    private MinioClient minioClient;

    @Resource
    private MinioConfiguration minioConfiguration;

    @PostMapping("/upload")
    public String upload(@RequestParam("file") MultipartFile file) throws Exception {
        String path = UUID.randomUUID() + file.getOriginalFilename();
        minioClient.putObject(PutObjectArgs.builder()
                .bucket(minioConfiguration.getBucket())
                .object(path)
                .stream(file.getInputStream(), file.getSize(), -1)
                .contentType(file.getContentType())
                .build());
        return String.format("%s/%s/%s",
                minioConfiguration.getEndpoint(),
                minioConfiguration.getBucket(),
                path);
    }
}
```

> 来源：鱼皮·编程导航 / codefather
