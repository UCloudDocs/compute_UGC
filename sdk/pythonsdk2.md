

# 任务例2 MapReduce

本文档将介绍如何使用Python SDK来完成一个任务，任务目标是统计一篇文章的每个单词出现的总次数。

我们将制作两个镜像，一个镜像名为mapper，用于统计每篇文章中每个单词出现的次数，另外一个镜像名为reducer，用于归总计算mapper任务返回的值。

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

## 三、制作并上传两个docker镜像

1.mapper.py脚本如下：

    #!usr/bin/env python
    import sys
    words=sys.stdin.read()
    r_words=words.replace('hate','like')
    with open('/tmp/result','w')  as f:
        f.write(str(r_words))
        f.close()

2.reducer.py脚本如下：

    import sys
    from collections import defaultdict
    
    content = sys.stdin.read()
    words=content.split()
    wordmap =  defaultdict(int)
    for word in words:
        wordmap[word] += 1
    with open('/tmp/map_result','w') as r:
        for k,v in wordmap.items():
            r.write("%s\t%s\n"%(k, v))

2.制作mapper镜像的dockerfile如下，请注意docker的版本应为1.6以上,制作reducer镜像的dokcerfile时，将下文的mapper.py替换为reducer.py即可。

    FROM centos:latest
    COPY ./mapper.py /usr/bin/mapper.py
    RUN chmod +x /usr/bin/mappper.py
    RUN echo "export TERM=xterm" >> /root/.bashrc
    CMD /usr/bin/python /usr/bin/mapper.py

3.登录镜像仓库,UserName和MyPassword分别替换为您在UCloud的登录邮箱及登录密码

    docker login -u UserName -p MyPassword cn-bj2.ugchub.service.ucloud.cn

4.分别制作reducer和mapper镜像，请注意将mytestbucket换成您自己创建的仓库名。

    docker build -t cn-bj2.ugchub.service.ucloud.cn/mytestbucket/mapper:first .
    docker build -t cn-bj2.ugchub.service.ucloud.cn/mytestbucket/reducer:first .

5.将镜像上传至通用计算的docker仓库。

    docker push cn-bj2.ugchub.service.ucloud.cn/mytestbucket/reducer:first
    docker push cn-bj2.ugchub.service.ucloud.cn/mytestbucket/mapper:first

## 四、使用SDK提交任务

1.修改example\_SubmitTaskAndGetResult.py文件，执行三次map任务，用于处理三个不同的文件：

    #!/usr/bin/env python
    # -*- coding: utf-8 -*-
    from sdk import apiInterface
    from sdk import tokenManager
    import json
    import time
    import tarfile, io
    import threading
    
    Images="cn-bj2.ugchub.service.ucloud.cn/kunkka/mapper:fifth"
    Input_datas=[open('/tmp/map_result','r'),open('/tmp/map_result2','r'),open('/tmp/map_result3','r')]
    
    def untarbytes(data):
        tar = tarfile.open(fileobj=io.BytesIO(data))
        for member in tar.getmembers():
            f = tar.extractfile(member)
            print f.name
            print f.read()
    
    def submit(token,Input_data):
        response = apiInterface.SubmitTask(ImageName=Images, AccessToken=token, OutputDir="/tmp",
        OutputFileName="map_result", TaskType="Sync", TaskName="testsync", Data=Input_data)
    
        if isinstance(response,dict):
            print "submit sync task fail" + response["Message"]
        else:
            print "submit sync task success:"
            print "untarbytes:"
            untarbytes(response)
            with open('/tmp/count','ab+') as c:
                c.write(response)
            print "write success"
    
    
    def run(token):
        threads = []
        for Input_data in Input_datas:
            threads.append(threading.Thread(target = submit, args = (token, Input_data)))
        for thread in threads:
            thread.start()
        for thread in threads:
            thread.join()
    
    if __name__=='__main__':
         tokenManager = tokenManager.TokenManager()
         token = tokenManager.getToken()
    
    print "example 1 submit mapper task"
    
    run(token)

2.执行脚本，分别提交了mapper任务，并返回了计算结果。

    [root@10-9-34-234 pythonSdk]# python t.py
    get :https://api.ucloud.cn?Action=GetAccessToken&PublicKey=ucloudzhangpengbo%40ucloud.cn1435155439000280342630&Region=cn-bj2&ExpireIn=7200&Signature=6154d4db58d8fc48d02ca95066b9f0a4f95e711e
    get token success : bfdba7e4-2761-4426-bd0e-d8c58c4e5a32
    TokenManager Starting with tokenRefreshTime : 6900
     example 1 submit mapper task
     post :http://api.ugc.service.ucloud.cn?AccessToken=bfdba7e4-2761-4426-bd0e-d8c58c4e5a32&Region=cn-bj2&Signature=6d5c2e1b9a6baeb6690d892fb980221e9a75a088&OutputFileName=map_result&PublicKey=ucloudzhangpengbo%40ucloud.cn1435155439000280342630&ImageName=cn-bj2.ugchub.service.ucloud.cn%2Fkunkka%2Fmapper%3Afifth&TaskType=Sync&Action=SubmitTask&TaskName=testsync&OutputDir=%2Ftmp
      post :http://api.ugc.service.ucloud.cn?AccessToken=bfdba7e4-2761-4426-bd0e-d8c58c4e5a32&Region=cn-bj2&Signature=6d5c2e1b9a6baeb6690d892fb980221e9a75a088&OutputFileName=map_result&PublicKey=ucloudzhangpengbo%40ucloud.cn1435155439000280342630&ImageName=cn-bj2.ugchub.service.ucloud.cn%2Fkunkka%2Fmapper%3Afifth&TaskType=Sync&Action=SubmitTask&TaskName=testsync&OutputDir=%2Ftmp
      post :http://api.ugc.service.ucloud.cn?AccessToken=bfdba7e4-2761-4426-bd0e-d8c58c4e5a32&Region=cn-bj2&Signature=6d5c2e1b9a6baeb6690d892fb980221e9a75a088&OutputFileName=map_result&PublicKey=ucloudzhangpengbo%40ucloud.cn1435155439000280342630&ImageName=cn-bj2.ugchub.service.ucloud.cn%2Fkunkka%2Fmapper%3Afifth&TaskType=Sync&Action=SubmitTask&TaskName=testsync&OutputDir=%2Ftmp
     submit map task success

3.执行example\_SubmitTaskAndGetResult.py，将Testimagesname、Outputfilename、Data做相应修改即可。

    #!/usr/bin/env python
    # -*- coding: utf-8 -*-
    from sdk import apiInterface
    from sdk import tokenManager
    import json
    import time
    import tarfile, io
    
    TestImageName="cn-bj2.ugchub.service.ucloud.cn/kunkka/reducer:first"
    
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
                   OutputDir="/tmp", OutputFileName="reducer_result", 
                   TaskType="Sync", TaskName="testsync", Data=open('/tmp/result','r'))
    
    if isinstance(response,dict):
       print "submit sync task fail" + response["Message"]
    else:
       print "submit sync task success:" 
       print response 
       print "untarbytes:" 
       untarbytes(response) 
       print ""
