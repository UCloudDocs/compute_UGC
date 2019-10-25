

## 镜像管理

### 创建镜像仓库

登录控制台后选择通用计算系统-\>镜像管理，选择“创建镜像仓库”。

![](/images/ugc_list_repo.png)

在“创建镜像仓库”弹窗页填写要创建的仓库名称。

![](/images/ugc_create_repo.png)

确定后创建成功，在列表中可以看到刚刚创建的仓库。

![](/images/ugc_create_result.png)

### 上传镜像

##### 1\. 登录镜像仓库

    $ docker login -u UCloudUserName -p MyPassword cn-bj2.ugchub.service.ucloud.cn

##### 2\. 制作镜像

准备运行算法脚本或文件，这里我们以一个简单的python脚本为例，完成在标准输出中输出hello
world，在标准错误中输出用户参数，同时在文件内容中返回用户post数据的任务。

Python脚本helloworld.py如下：

    #!/usr/bin/env python
    import sys
    print("hello world")
    body = sys.stdin.read()
    sys.stderr.write(str(sys.argv[1:]))
    with open("/tmp/result", "w") as f:
        f.write(body)

Dockerfile 如下，注意 docker版本应为1.9.1 以上

    FROM centos:latest
    COPY ./helloworld.py /usr/bin/helloworld.py
    RUN chmod +x /usr/bin/helloworld.py
    RUN echo "export TERM=xterm" >> /root/.bashrc
    ENTRYPOINT ["/usr/bin/helloworld.py"]

创建镜像，mytestbucket为用户创建的bucket名称，helloworld为镜像名，版本号为first

    docker build -t cn-bj2.ugchub.service.ucloud.cn/mytestbucket/helloworld:first .
    Sending build context to Docker daemon 11.26 kB
    Step 1 : FROM centos:latest
    ---> 970633036444
    Step 2 : COPY ./helloworld.py /usr/bin/helloworld.py
    ---> Using cache
    ---> eec72523d3c1
    Step 3 : RUN chmod +x /usr/bin/helloworld.py
    ---> Running in 9c19ab579b4e
    ---> df3eee0a73b6
    Removing intermediate container 9c19ab579b4e
    Step 4 : RUN echo "export TERM=xterm" >> /root/.bashrc
    ---> Running in 8882c0a146d2
    ---> 9612b71f4a14
    Removing intermediate container 8882c0a146d2
    Step 5 : ENTRYPOINT /usr/bin/helloworld.py
    ---> Running in e65a4b905b43
    ---> a041f6384047
    Removing intermediate container e65a4b905b43

##### 3\. 上传镜像

    docker push cn-bj2.ugchub.service.ucloud.cn/mytestbucket/helloworld:first

##### 4\. 查看镜像详情

镜像管理中点击对应的仓库可以显示仓库中推送镜像的详情。

![](/images/bucketdetail_new.png)

### 告警管理

登录控制台后，选择通用计算-\>镜像管理-\>选择某个私有仓库

![](/images/alert_manager.png)

点击“告警管理”，选择对应的通用计算告警模板

![](/images/alert_template.png)

编辑告警模板，可以选择监控-\>点击“创建模板”

![](/images/alert_template_edit.png)

创建通用计算的告警模板

![](/images/create_alert_template.png)

选择需要的告警规则并编辑阈值等

![](/images/chose_alert_rule.png)

### 数据视图

登录控制台后，选择通用计算-\>镜像管理-\>选择某个私有仓库

![](/images/data_view.png)

点击“数据视图”

![](/images/data_view_detail.png)
