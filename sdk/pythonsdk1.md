

# 任务例1 字符串替换

为了方便Python开发人员更高效地使用通用计算，我们提供了Python SDK，Python版本为2.x。本文档将介绍如何使用Python
SDK来完成一个任务，任务目标是将文章内所有的“hate”替换为“like”。

备注：通用计算不支持外网调用，您必须申请一台北京二地域的云主机。

## 一、准备工作

下载Python SDK，SDK的Python版本为2.x。

    $ git clone https://www.github.com/ucloud/ugc-sdk.git

## 二、使用SDK创建镜像仓库

1.打开config.py文件，填入UCloud账号的公私钥,代码如下：

    #-*- encoding: utf-8 -*-
    #配置公私钥
    PRIVATE_KEY = ""   # 控制台获取  # 项目ID 请在Dashbord 上获取，通用计算项目目前不区分项目，可不填
    PROJECT_ID  = ""
    
    # API地址
    COMMON_API_URL    = "https://api.ucloud.cn" #管理API使用的外网域名
    TASK_API_URL      = "http://api.ugc.service.ucloud.cn" # 提交任务相API使用的内网域名
    
    # region参数
    REGION      = "cn-bj2"  # 区域列表,参见https://docs.ucloud.cn/api-docs-az/summary/regionlist.htmltt
    
    #token默认过期时间
    TOKEN_EXPIRE_TIME=7200

2.执行CreateDockerImageBucket.py，即可创建一个mytestbucket的镜像仓库。

    #!/usr/bin/env python
    # -*- coding: utf-8 -*-
    from sdk import apiInterface
    import json
    if __name__=='__main__':
        print "example 1 CreateDockerImageBucket 1"
        response = apiInterface.CreateDockerImageBucket(BucketName="mytestbucket")
        print json.dumps(response, sort_keys=True, indent=4, separators=(',', ': '))
        print ""

## 三、制作并上传docker镜像

1.python脚本如下：

    #!usr/bin/env python
    import sys
    words=sys.stdin.read()
    r_words=words.replace('hate','like')
    with open('/tmp/result','w')  as f:
        f.write(str(r_words))
        f.close()

2.dockerfile如下，请注意docker的版本应为1.6以上。

    FROM centos:latest
    COPY ./replace.py /usr/bin/replace.py
    RUN echo "export TERM=xterm" >> /root/.bashrc
    CMD /usr/bin/python /usr/bin/replace.py

3.登录镜像仓库,UserName和MyPassword分别替换为您在UCloud的登录邮箱及登录密码

    docker login -u UserName -p MyPassword cn-bj2.ugchub.service.ucloud.cn

4.制作镜像，请注意将mytestbucket换成您自己创建的仓库名。

    docker build -t cn-bj2.ugchub.service.ucloud.cn/mytestbucket/replace:first .

5.将镜像上传至通用计算的docker仓库。

    docker push cn-bj2.ugchub.service.ucloud.cn/mytestbucket/replace:first

## 四、使用SDK提交任务

1.打开example\_SubmitTaskAndGetResult.py文件，修改启动镜像名、date、输出目录、输出文件参数值。代码示例如下(去掉了异步任务的调用方法)：

    #!/usr/bin/env python
    # -*- coding: utf-8 -*-
    from sdk import apiInterface
    from sdk import tokenManager
    import json
    import time
    import tarfile, io
    
     TestImageName="cn-bj2.ugchub.service.ucloud.cn/kunkka/replace:3"
    
    def untarbytes(data):
    tar = tarfile.open(fileobj=io.BytesIO(data))
    for member in tar.getmembers():
        f = tar.extractfile(member)
        print f.name
        print f.read()
    
    if __name__=='__main__':
        tokenManager = tokenManager.TokenManager()
        token = tokenManager.getToken()
        print "example 1 submit sync task"
        response = apiInterface.SubmitTask(ImageName=TestImageName, AccessToken=token,
        OutputDir="/tmp", OutputFileName="result", TaskType="Sync", TaskName="testsync", Data="lilei hate lucy")
    
     if isinstance(response,dict):
        print "submit sync task fail" + response["Message"]
    else:
        print "submit sync task success:"
        print response
        print "untarbytes:"
        untarbytes(response)
    print ""

2.执行example\_SubmitTaskAndGetResult.py文件，便可以看到返回结果,输出结果已经更改为lilei like
lucy。

    [root@10-9-34-234 pythonSdk]# python example_SubmitTaskAndGetResult.py
    get :https://api.ucloud.cn?Action=GetAccessToken&PublicKey=ucloudzhangpengbo%40ucloud.cn1435155439000280342630&Region=cn-bj2&ExpireIn=7200&Signature=6154d4db58d8fc48d02ca95066b9f0a4f95e711e
    get token success : 8dad4bc0-8b12-425b-82ee-b734357ef035
    TokenManager Starting with tokenRefreshTime : 6900
    example 1 submit sync task
    post :http://api.ugc.service.ucloud.cn?AccessToken=8dad4bc0-8b12-425b-82ee-b734357ef035&Region=cn-bj2&Signature=00d3c8d5402416492afa53553af84565a6ce1cb6&OutputFileName=result&PublicKey=ucloudzhangpengbo%40ucloud.cn1435155439000280342630&ImageName=cn-bj2.ugchub.service.ucloud.cn%2Fkunkka%2Freplace%3Afour&TaskType=Sync&Action=SubmitTask&TaskName=testsync&OutputDir=%2Ftmp
    submit sync task success:
    /tmp/result0000600000000000000000000000001712750555426011305 ustar0000000000000000lilei like lucy
    untarbytes:
    /tmp/result
    lilei like lucy
