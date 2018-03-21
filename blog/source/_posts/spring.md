---
toc: true
title: spring boot + mongoDB + heroku 构建一个小型文件储存系统
date: 2018-01-19 00:12:00
tags:
 - spring boot
 - mongoDB
 - heroku
categories: java
---

## Introduction：

### 需求：

因为自己建造了一个blog ，在找国外的file system 可以给自己的blog 插图的。找来找去没找到一个合适的。就打算自己写一个，主要自己能管理自己的图片啊，外部网站啊什么的。

初步暂定就是用spring boot 作为服务器端。然后在考虑储存图片的时候就找了好多方法，当初脑子里最直接的想法是Amazon 的S3。 但是我真的不想把自己的信用卡信息交给AWS 来管理。总有点不信任AWS 的感觉。 虽然马上要去Amazon 搬砖。以后肯定会用到S3的技术，我就没必要再玩S3了。 后来就查到了mongoDB 可以实现储存16MB的文件，决定用mongoDB 实现这个service。 

本文讲介绍如何通过用spring boot 加云服务上的mongoDB（[mlab](https://mlab.com/)） 并deploy到[heroku](https://heroku.com) 实现一个有500MB的文件储存系统。

### Spring boot:

[Spring boot](https://projects.spring.io/spring-boot/) 是一款轻量级框架，可以大部分减少前期项目配置spring 的动作。很多微服务都是通过spring boot实现的！

Spring Boot的特性有以下几条：

- 创建独立Spring应用程序
- 嵌入式Tomcat，Jetty容器，无需部署WAR包
- 简化Maven及Gradle配置
- 尽可能的自动化配置Spring
- 直接植入产品环境下的实用功能，比如度量指标、健康检查及扩展配置等
- 无需代码生成及XML配置

如果对spring boot 不是很熟悉的同学，推荐：

https://github.com/tengj/SpringBootDemo 这个人的github. 上面有非常不错的spring boot 教程！

### MongoDB：

[MongoDB](https://www.mongodb.com/) 是一款NoSQL 文档型数据库，开源，是当前nosql 中最热门的一种。它是用C++ 编写的。有很多种语言的API。

MongoDB 主要特点：

- 高性能、易部署、易使用，存储数据非常方便。
- 面向集合存储，易存储对象类型的数据。 
- 支持动态查询。 
- 支持完全索引，包含内部对象，支持查询。 
- 支持复制和故障恢复。 
- 使用高效的二进制数据存储，包括大型对象
- 自动处理碎片，以支持云计算层次的扩展性。
- 文件存储格式为BSON（一种JSON的扩展，Binary JSON）。


在这里MongoDB 储存文件最多不超过16MB. 这个也可以在spring-boot 里面配置每次上传文件的大小， 在这我们只设置了3M 大小的

因为我们在这把这个服务主要当作储存我们blog 图片的服务器。

### Heroku：

[Heroku](https://heroku.com) 是一个自动化部署的平台，可以链接你的github 账号里的project repo， 只要你的项目有更新，他就会自动deploy，而且还是免费的。 感觉比AWS elastic beanstalk 好用些。

免费的plan 是512RAM。可以有5个free APP。 还是蛮不错的。可以让你在dynos一个月免费运行550小时。

具体不多说。 各位客官自行研究吧。

## 准备：

- Git
- Heroku Account and Heroku CLI
- maven
- mlab account - mongoDB uri

Git、 maven、不多说了

### 配置heroku CLI

ENV: Windows bash-Ubuntu 16.04
```bash
$ wget -qO- https://cli-assets.heroku.com/install-ubuntu.sh | sh
```

登录heroku

```bash
$ heroku login
```

## 建立一个能运行在Heroku的Spring boot App:

```bash
$ git clone https://github.com/heroku/java-getting-started.git
$ cd java-getting-started
$ mvn install
```

### 在本地上运行

```bash
$ heroku local:start
```

那么现在你就可以访问[localhost:5000](http://localhost:5000/) 去查看你已经建好的example-web app

### Deploy 到Heroku

```bash
$ heroku create #建立一个heroku app，只需要建立一次
$ git add .
$ git commit -m "update"
$ git push heroku master #发送到heroku master
$ heroku open
```

访问heroku 给出的link 就能达到你的网站了。

### Procfile:

这个文件是Deploy 到Heroku 上必须要有的。你可以参考[heroku Procfile](https://devcenter.heroku.com/articles/procfile)官网配置，在这里我们直接用heroku 的example git repo 就不用自己配置了。

## 建立MongoDB(mlab)

访问[mlab](https://mlab.com) 建立一个database as a service.

首先注册。。。

然后点击create New

![](https://floating-waters-18924.herokuapp.com/files/5a62394b234fea000498d033)

然后选择一个Cloud service 作为你的DB host， 推荐Amazon 爸爸。

![](https://floating-waters-18924.herokuapp.com/files/5a623998234fea000498d034)

然后建立一个user，得到你的mongoDB URI：

![](https://floating-waters-18924.herokuapp.com/files/5a623a7a234fea000498d035)

好了，你的mongoDB 就建好了，虽然只有500MB啊 哈哈哈。

## 代码：

note：以下代码参考了这个大牛的[github](https://github.com/waylau/mongodb-file-server)

我的代码在我自己的[github](https://github.com/eeycl3/mongoDbFileServer)

### 部分maven 配置 pom.xml 

```xml
    <!--Spring boot mongoDB 配置-->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-data-mongodb</artifactId>
      <version>1.5.2.RELEASE</version>
    </dependency>
    
    <!--embedded mongoDB 配置 for local-->
    <dependency>
      <groupId>de.flapdoodle.embed</groupId>
      <artifactId>de.flapdoodle.embed.mongo</artifactId>
      <version>2.0.0</version>
      <scope>test</scope>
    </dependency>
```

### Spring 配置

```latex
# connection
spring.datasource.hikari.connection-timeout=30000
spring.datasource.hikari.maximum-pool-size=10

#logger
logging.level.org.springframework=INFO
spring.profiles.active=production

#server port
server.port=${PORT:8080}

# Thymeleaf
spring.thymeleaf.encoding=UTF-8
spring.thymeleaf.cache=false
spring.thymeleaf.mode=HTML5

# limit upload file size
spring.http.multipart.max-file-size=3MB
spring.http.multipart.max-request-size=3MB

# independent MongoDB server 
# set mongoDB URI as enviromental var
spring.data.mongodb.uri=${MONGODB_URI}
```

在这里我们把mongoDB uri 设置为环境变量，你可以在heroku 的dashboard 里配置环境变量。

### Model

```java
package com.blogpic.fileserver.model;

import java.util.Date;
import lombok.Data;
import lombok.RequiredArgsConstructor;
import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.mapping.Document;

@Data
@RequiredArgsConstructor
@Document
public class File {
	@Id  // primary key
	private String id;
    private String name; // file name
    private String contentType; // file type
    private long size;
    private Date uploadDate;
    private String md5;
    private byte[] content; // content
    private String path; // path

    public File(String name, String contentType, long size, byte[] content) {
    	this.name = name;
    	this.contentType = contentType;
    	this.size = size;
    	this.uploadDate = new Date();
    	this.content = content;
    }
}
```

### Service

Interface:

```java
package com.blogpic.fileserver.service;

import java.util.List;

import com.blogpic.fileserver.model.File;

public interface FileService {
	/**
	 * 保存文件
	 * @param file
	 * @return
	 */
	File saveFile(File file);
	
	/**
	 * 删除文件
	 * @param id
	 * @return
	 */
	void removeFile(String id);
	
	/**
	 * 根据id获取文件
	 * @param id
	 * @return
	 */
	File getFileById(String id);

	/**
	 * 分页查询，按上传时间降序
	 * @param pageIndex
	 * @param pageSize
	 * @return
	 */
	List<File> listFilesByPage(int pageIndex, int pageSize);
}

```

impl:

```java
package com.blogpic.fileserver.service;

import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.PageRequest;
import org.springframework.data.domain.Pageable;
import org.springframework.data.domain.Sort;
import org.springframework.data.domain.Sort.Direction;
import org.springframework.stereotype.Service;
import com.blogpic.fileserver.model.File;
import com.blogpic.fileserver.repository.FileRepository;

@Service
public class FileServiceImpl implements FileService {
   
   @Autowired
   public FileRepository fileRepository;

   @Override
   public File saveFile(File file) {
      return fileRepository.save(file);
   }

   @Override
   public void removeFile(String id) {
      fileRepository.delete(id);
   }

   @Override
   public File getFileById(String id) {
      return fileRepository.findOne(id);
   }

   @Override
   public List<File> listFilesByPage(int pageIndex, int pageSize) {
      Page<File> page = null;
      List<File> list = null;
      
      Sort sort = new Sort(Direction.DESC,"uploadDate"); 
      Pageable pageable = new PageRequest(pageIndex, pageSize, sort);
      
      page = fileRepository.findAll(pageable);
      list = page.getContent();
      return list;
   }
}
```

### Repo(Dao)

```java
package com.blogpic.fileserver.repository;

import com.blogpic.fileserver.model.File;
import org.springframework.data.mongodb.repository.MongoRepository;

public interface FileRepository extends MongoRepository<File, String> {
}
```

### Controller

```java
package com.blogpic.fileserver.controller;

import java.io.IOException;
import java.security.NoSuchAlgorithmException;
import java.util.List;

import com.blogpic.fileserver.model.File;
import com.blogpic.fileserver.service.FileService;
import com.blogpic.fileserver.util.MD5Util;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.http.HttpHeaders;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.CrossOrigin;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.multipart.MultipartFile;
import org.springframework.web.servlet.mvc.support.RedirectAttributes;

@CrossOrigin(origins = "*", maxAge = 3600)  // 允许所有域名访问
@Controller
public class FileController {

    @Autowired
    private FileService fileService;
    
    @Value("${server.port}")
    private String serverPort;
    
    @RequestMapping(value = "/")
    public String index(Model model) {
        model.addAttribute("files", fileService.listFilesByPage(0,20)); 
        return "index";
    }

    /**
     * 分页查询文件
     * @param pageIndex
     * @param pageSize
     * @return
     */
	@GetMapping("files/{pageIndex}/{pageSize}")
    @ResponseBody
	public List<File> listFilesByPage(@PathVariable int pageIndex, @PathVariable int pageSize){
		return fileService.listFilesByPage(pageIndex, pageSize);
	}
			
    /**
     * 获取文件片信息
     * @param id
     * @return
     */
    @GetMapping("files/{id}")
    @ResponseBody
    public ResponseEntity<Object> serveFile(@PathVariable String id) {

        File file = fileService.getFileById(id);

        if (file != null) {
            return ResponseEntity
                    .ok()
                    .header(HttpHeaders.CONTENT_DISPOSITION, "attachment; fileName=\"" + file.getName() + "\"")
                    .header(HttpHeaders.CONTENT_TYPE, "application/octet-stream" )
                    .header(HttpHeaders.CONTENT_LENGTH, file.getSize()+"")
                    .header("Connection",  "close") 
                    .body( file.getContent());
        } else {
            return ResponseEntity.status(HttpStatus.NOT_FOUND).body("File was not fount");
        }

    }
    
    /**
     * 在线显示文件
     * @param id
     * @return
     */
    @GetMapping("/view/{id}")
    @ResponseBody
    public ResponseEntity<Object> serveFileOnline(@PathVariable String id) {

        File file = fileService.getFileById(id);

        if (file != null) {
            return ResponseEntity
                    .ok()
                    .header(HttpHeaders.CONTENT_DISPOSITION, "fileName=\"" + file.getName() + "\"")
                    .header(HttpHeaders.CONTENT_TYPE, file.getContentType() )
                    .header(HttpHeaders.CONTENT_LENGTH, file.getSize()+"")
                    .header("Connection",  "close") 
                    .body( file.getContent());
        } else {
            return ResponseEntity.status(HttpStatus.NOT_FOUND).body("File was not fount");
        }

    }
    
    /**
     * upload
     * @param file
     * @param redirectAttributes
     * @return
     */
    @PostMapping("/")
    public String handleFileUpload(@RequestParam("file") MultipartFile file,
                                   RedirectAttributes redirectAttributes) {

        try {
        	File f = new File(file.getOriginalFilename(),  file.getContentType(), file.getSize(), file.getBytes());
        	f.setMd5( MD5Util.getMD5(file.getInputStream()) );
        	fileService.saveFile(f);
        } catch (IOException | NoSuchAlgorithmException ex) {
            ex.printStackTrace();
            redirectAttributes.addFlashAttribute("message",
                    "Your " + file.getOriginalFilename() + " is wrong!");
            return "redirect:/";
        }

        redirectAttributes.addFlashAttribute("message",
                "You successfully uploaded " + file.getOriginalFilename() + "!");

        return "redirect:/";
    }
 
    /**
     * upload file
     * @param file
     * @return
     */
    @PostMapping("/upload")
    @ResponseBody
    public ResponseEntity<String> handleFileUpload(@RequestParam("file") MultipartFile file) {
    	File returnFile = null;
        try {
        	File f = new File(file.getOriginalFilename(),  file.getContentType(), file.getSize(),file.getBytes());
        	f.setMd5( MD5Util.getMD5(file.getInputStream()) );
        	returnFile = fileService.saveFile(f);
        	String path = "//"+ "localhost" + ":" + serverPort + "/view/"+returnFile.getId();
        	return ResponseEntity.status(HttpStatus.OK).body(path);
 
        } catch (IOException | NoSuchAlgorithmException ex) {
            ex.printStackTrace();
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(ex.getMessage());
        }
 
    }
    
    /**
     * delete file
     * @param id
     * @param redirectAttributes
     * @return
     */
    @PostMapping("/delete")
    public String deleteFile(@RequestParam String id, RedirectAttributes redirectAttributes) {
    	try {
			fileService.removeFile(id);
		} catch (Exception e) {
            redirectAttributes.addFlashAttribute("message", "Fail to delete file with id: " + id);
            return "redirect:/";
		}
        redirectAttributes.addFlashAttribute("message", "Succeed to delete file with id: " + id);
        return "redirect:/";
    }
}
```

## Final
![](https://floating-waters-18924.herokuapp.com/files/5a6261715c2f8e0004eff88f)


