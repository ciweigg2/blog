cover: http://ciwei2.cn-sh2.ufileos.com/51.jpg
title: ipfs分布式文件存储系统
date: 2019-05-03T09:08:12.048Z
tags: [ipfs]
categories: [综合]
---
### 安装ipfs

```java
docker run -d --name ipfs_host -e IPFS_PROFILE=server -v /root/ipfs/export:/export -v /root/ipfs/data:/data/ipfs -p 4001:4001 -p 8023:8080 -p 5001:5001 ipfs/go-ipfs:latest
```

<!--more-->

![](/images/ipfswebuis.png)

* 界面会有提示执行跨域访问 然后重启ipfs_host

```java
docker exec ipfs_host ipfs config --json API.HTTPHeaders.Access-Control-Allow-Origin '["http://192.168.15.59:5001", "http://127.0.0.1:5001", "https://webui.ipfs.io"]'

docker exec ipfs_host ipfs config --json API.HTTPHeaders.Access-Control-Allow-Methods '["PUT", "GET", "POST"]'

docker stop ipfs_host

docker start ipfs_host
```

* 首次进入需要设置API地址/ip4/192.168.15.59/tcp/5001 点击提交就可以了呀

```java
http://192.168.15.59:5001/webui

界面中的settings需要配置网关 因为上面8080端口映射了8023所以需要配置8023端口呀

"Gateway": "/ip4/192.168.15.59/tcp/8023"
```

* 具体可以参考：https://github.com/ipfs/go-ipfs

### 使用命令上传文件

首先上传图片到/root/ipfs/export目录下面 然后执行下面的语句
docker exec ipfs_host ipfs add -r /export/0.jpg
会产生两个哈希 第一个对应图片 第二个对应文件夹 在界面上可以搜索的

```java
1017.66 KiB / 1017.66 KiB  100.00%added QmV6kNiRmdfYkXP75JrgyXyfSwL7EvMMyoxo9g3e7kMk7J export/0.jpg
added QmRY2YPjNhzR5fqej9mEyDjcXpwCGpB8EZUZNQvxAo8fvZ export
```

访问上传的图片：http://192.168.15.59:8023/ipfs/QmNWt8iJDUvEVQMV9C3Gp522VAxb19HUYTbWiggvVV4DYX

如果删除了ipfs容器 但是持久化数据还在是起不来的 需要将vi /root/ipfs/data/config 里面的"Gateway": "/ip4/192.168.15.59/tcp/8023"改成"Gateway": "/ip4/0.0.0.0/tcp/8080" 启动后再去界面settings改成"Gateway": "/ip4/192.168.15.59/tcp/8023"

### java对接

https://github.com/ipfs/java-ipfs-http-client

```java
    @Autowired
    private IPFS ipfs;

    /**
     * 初始化ipfs分布式文件系统
     * <p>
     * https://github.com/ipfs/java-ipfs-http-client
     */
    @Bean
    public IPFS init() {
        IPFS ipfs = new IPFS("/ip4/192.168.15.59/tcp/5001");
        return ipfs;
    }

    @GetMapping("/uploudFile")
    public String ipfsUploudFile() throws IOException {
        //上传文件file类型
        NamedStreamable.FileWrapper file = new NamedStreamable.FileWrapper(new File("C:\\Users\\Administrator\\Pictures\\lovewallpaper\\1516979866320_feeds_152999926_0-0.jpeg"));
        MerkleNode addResult = ipfs.add(file).get(0);
        log.info(addResult.hash.toString());
        return "上传成功呀：" + addResult.hash.toString();
    }

    @GetMapping("/uploudByte")
    public String ipfsUploudByte() throws IOException {
        //上传文件byte[]类型 可以是文件也可以是内容呀
        NamedStreamable.ByteArrayWrapper file = new NamedStreamable.ByteArrayWrapper(image2byte("C:\\Users\\Administrator\\Pictures\\lovewallpaper\\927_2016050942101789.jpg"));
        MerkleNode addResult = ipfs.add(file).get(0);
        log.info(addResult.hash.toString());
        return "上传成功呀：" + addResult.hash.toString();
    }

    @GetMapping("/get")
    public String get() throws IOException {
        //获取文件
        Multihash filePointer = Multihash.fromBase58("QmNWt8iJDUvEVQMV9C3Gp522VAxb19HUYTbWiggvVV4DYX");
        byte[] fileContents = ipfs.cat(filePointer);
        return "文件二进制：" + fileContents;
    }

    //图片到byte数组
    public byte[] image2byte(String path) {
        byte[] data = null;
        FileImageInputStream input = null;
        try {
            input = new FileImageInputStream(new File(path));
            ByteArrayOutputStream output = new ByteArrayOutputStream();
            byte[] buf = new byte[1024];
            int numBytesRead = 0;
            while ((numBytesRead = input.read(buf)) != -1) {
                output.write(buf, 0, numBytesRead);
            }
            data = output.toByteArray();
            output.close();
            input.close();
        } catch (FileNotFoundException ex1) {
            ex1.printStackTrace();
        } catch (IOException ex1) {
            ex1.printStackTrace();
        }
        return data;
    }
```