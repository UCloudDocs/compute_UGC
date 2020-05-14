

# 任务例3 单词统计

本文档将介绍如何使用Golang
SDK来完成一个任务，任务目标是统计文本inputfile1内所有的所有单词出现次数，统计结果保存到文本outputfile1。考虑到很多ugc用户使用ufile存取文件，本例将适当描述ufile存取文件相关操作。任务处理的整体流程为：从Ufile下载输入文件，提交任务到UGC，UGC执行任务，UGC返回结果文件，结果文件上传到Ufile。

备注：通用计算支持外网调用，但外网带宽有限，强烈建议您申请一台北京二地域的云主机使用内网调用。本教程所有代码可以在github获取：<https://github.com/ucloud/ugc-sdk/tree/master/golangSdk/example/example2>

## 一、准备工作

1.下载Golang SDK

    $ git clone https://github.com/ucloud/ugc-sdk.git

２.下载Ufile存取文件相关工具，本例子下载linux工具，其它系统参考https://docs.ucloud.cn/ufile/tools/tools_file

    $ wget　http://tools.ufile.ucloud.com.cn/bucketmgr-linux64.tar.gz
    $ wget　http://tools.ufile.ucloud.com.cn/filemgr-linux64.tar.gz

3.配置Ufile工具，详细参考https://docs.ucloud.cn/ufile/tools/tools_file

## 二、使用SDK创建镜像仓库

替换PUBLIC\_KEY和PRIVATE\_KEY为您自己的，编译并执行CreateDockerImageBucket.go，即可创建一个mytestbucket的镜像仓库。密钥获取：<https://console.ucloud.cn/apikey>　，本SDK已实现签名算法，其它语言可参考实现签名算法：<https://docs.ucloud.cn/api/summary/signature>

    //CreateDockerImageBucket.go
    package main
    
    import (
        "fmt"
        "ugc-sdk/golangSdk/golandSdk"
    )
    
    //PUBLIC_KEY和PRIVATE_KEY替换为您的
    var PUBLIC_KEY string = ""
    var PRIVATE_KEY string = ""
    
    func createDockerImageBucket() {
        var request ugcapi.CreateDockerImageBucketRequest
        request.Region = "cn-bj2"
        request.BucketName = "testBucketName"
        request.PublicKey = PUBLIC_KEY
        request.PrivateKey = PRIVATE_KEY
    
        rsp, err := ugcapi.CreateDockerImageBucket(request)
        if err != nil {
            fmt.Println("CreateDockerImageBucket", err)
            return
        } else {
            fmt.Println(rsp)
        }
        return
    }
    
    func main() {
        createDockerImageBucket()
        return
    }

## 三、制作并上传docker镜像

1.Golang代码主函数main.go如下，注意：输入数据必须从标准输入读取（用户SubmitTask
Post请求Body中的数据会提交到容器的Stdin），输出数据写入容器路径必须和SubmitTask中指定的路径一致（OutputDir和OutputFilename）

    package main
    
    import (
        "fmt"
        "io/ioutil"
        "os"
        "wordCount/logic"
    )
    
    func main() {
        //必须从标准输入读取数据（用户SubmitTask Post请求Body中的数据会提交到容器的Stdin）
        inputFileText, err := ioutil.ReadAll(os.Stdin)
        if err != nil {
            fmt.Print("ioutil.ReadAll error", err)
            return
        }
            //结果写入路径必须和SubmitTask中指定的路径一致（OutputDir和OutputFilename） 
        outputFile := "/tmp/result"　　
        //wordCount算法核心逻辑
        outputFileText := logic.CountTestBase(string(inputFileText))
        //结果写到Docker容器中相应的目录
        err = ioutil.WriteFile(outputFile, []byte(outputFileText), os.ModePerm)
        if err != nil {
            fmt.Print("ioutil.WriteFile error", err)
        }
        return
    }

2.dockerfile如下，请注意docker的版本应为1.9.1以上。

    FROM centos:latest
    COPY ./wordCount /usr/bin/wordCount
    RUN echo "export TERM=xterm" >> /root/.bashrc
    CMD /usr/bin/wordCount

3.登录镜像仓库,UserName和MyPassword分别替换为您在UCloud的登录邮箱及登录密码

    $ sudo docker login -u UserName -p MyPassword cn-bj2.ugchub.service.ucloud.cn

4.制作镜像，请注意将mytestbucket换成您自己创建的仓库名。

    $ sudo docker build -t cn-bj2.ugchub.service.ucloud.cn/mytestbucket/wordcount:first .

5.将镜像上传至通用计算的docker仓库。

    $ sudo docker push cn-bj2.ugchub.service.ucloud.cn/mytestbucket/wordcount:first

## 四、使用SDK提交任务

1.从Ufile下载文件：使用Ufile工具（也可使用Ufile SDK下载文件，Ufile
SDK使用参考https://docs.ucloud.cn/ufile/tools）下载待处理文件到本地（假设用户已上传待处理文件到Ufile）

    $ ./filemgr-linux64 --action download  --bucket testbucket --key inputfile1 --file inputfile1

2.提交任务到UGC：编译并执行SubmitTask.go，需要修改启动镜像名、公钥私钥、输出目录、输出文件参数值等。注意SubmitTask提交的数据必须通过Post请求的Body传入。

    //SubmitTask.go
    package main
    
    import (
        "encoding/json"
        "fmt"
        "io/ioutil"
        "ugc-sdk/golangSdk/golandSdk"
    )
    
    //PUBLIC_KEY和PRIVATE_KEY替换为你的
    var PUBLIC_KEY string = ""
    var PRIVATE_KEY string = ""
    var ACCESS_TOKEN string = ""
    
    func getAccessToken() {
        type AccessTokenRsp struct {
            AccessToken string
        }
        var request ugcapi.GetAccessTokenRequest
        request.Region = "cn-bj2"
        request.PublicKey = PUBLIC_KEY
        request.PrivateKey = PRIVATE_KEY
        rsp, err := ugcapi.GetAccessToken(request)
        if err != nil {
            fmt.Println("GetAccessToken failed ", err)
            return
        }
        bytebody := []byte(rsp)
        var atp AccessTokenRsp
        err = json.Unmarshal(bytebody, &atp)
        if err != nil {
            fmt.Println("json failed ", err)
            return
        }
        ACCESS_TOKEN = atp.AccessToken
    }
    
    func submitTask() {
        var request ugcapi.SubmitTaskRequest
        request.Region = "cn-bj2"
        //替换为您的镜像名
        request.ImageName = "cn-bj2.ugchub.service.ucloud.cn/mytestbucket/wordCount:first"
            //OutputDir和OutputFilename需要和容器中指定的路径相同
            //例如容器中指定输出路径为/tmp/result，则OutputDir为/tmp，OutputFilename为result。
        request.OutputDir = "/tmp"　　
        request.OutputFilename = "result"
        request.TaskName = "for_test"
        request.TaskType = "Sync"
        request.AccessToken = ACCESS_TOKEN
        //替换为你需要上传文件的路径，（上1步Ufile下载的文件路径）
        inputFilePath := "/yourPath/inputfile1"
        inputFile, err := ioutil.ReadFile(inputFilePath)
        //输入数据必须发送Post请求，inputFile通过Body传送
        rsp, err := ugcapi.SubmitTask(request, string(inputFile))
    
        if err != nil {
            fmt.Println("SubmitTask failed ", err)
            return
        } else {
            //返回的处理结果，保存为outputfile1
            err := ioutil.WriteFile("outputfile1", []byte(rsp), 0600)
            if err != nil {
                fmt.Println("submitTask ioutil.WriteFile() error ", err)
            }
        }
        return
    }
    
    func main() {
        getAccessToken()
        submitTask()
    }

3.UGC返回结果文件：通过执行SubmitTask，可以看到本地多了一个文件outputfile1，通过file
命令能看出这是个tar包，里面包含数据处理完的文件。解压tar包如下:

    $ tar xvf outputfile1
    outputfile1  tmp

./tmp/result文件如下(只展示部分结果)：

    程序执行：25ms
    文章总单词数：98238
    
    the: 5771, 5.87%
    and: 3346, 3.41%
    to: 2148, 2.19%
    of: 1795, 1.83%
    in: 1595, 1.62%
    i: 1513, 1.54%
    d: 1407, 1.43%
    a: 1373, 1.40%
    with: 1038, 1.06%
    my: 886, 0.90%
    his: 818, 0.83%
    he: 799, 0.81%
    is: 798, 0.81%
    s: 793, 0.81%

4.上传结果文件到Ufile，使用Ufile工具（也可使用Ufile SDK上传文件，Ufile
SDK使用参考https://docs.ucloud.cn/ufile/tools）上传处理好文件到Ufile。

    $ ./filemgr-linux64 --action put --bucket testbucket --key outputfile1 --file outputfile1
