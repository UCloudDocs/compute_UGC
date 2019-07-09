{{indexmenu_n>1010}}

## 任务管理

### 通过控制台提交异步任务

选择任务管理栏，点击提交任务

![](/images/ugc_list_task.png)

在提交任务弹窗中填写参数：

1\. 填写自定义的任务名称 2.
选择需要运行的镜像名（这里我们使用在6.1中制作的镜像cn-bj2.ugchub.service.ucloud.cn/mytestbucket/helloworld:first）
3.
在可选参数输出路径和输出文件中分别填写”/tmp”和”result”，这个我们制作的镜像算法的输出文件，运行完成后该输出文件将通过二进制数据流的形式被返回。
4. 点击提交任务执行

![](/images/submittask_new.png)

执行任务后发现任务列表中的任务处于运行状态（注意：页面提交的任务均为异步任务），稍后任务更新为成功状态。

![](/images/ugc_submit_result.png)

点击查看，检查任务运行详情：

可以在Stdout摘要中看到任务的输出摘要：hello world，在Stderr摘要中看到任务的输入参数。
（注意任务摘要只会截取镜像算法的Stdout和Stderr的部分报错，要获取完整的结果需要通过API调用获取）

![](/images/ugc_task_info.png)

### 通过API提交同步任务

这里演示通过shell命令演示通过API提交同步任务的过程。

##### 1\. 通过GetAccessToken API获取Token

注意填写正确的PublicKey和签名，签名算法见API文档概述。

![](/images/ugc_get_token.png)

##### 2\. 通过SubmitTask API提交任务

使用POST提交内容为“this is post data”

Content-Type设置为application/octet-stream

ImageName参数为用户提交的镜像名全称

TaskName为用户自定义任务名称，便于标记任务

AccessToken为上一步申请的Token参数

TaskType参数为Async表示异步任务

Cmd参数为运行镜像的输入参数，这里自定义为”thisiscmd”

OutputDir和OutputFileName参数为要输出的镜像文件内容

![](/images/ugc_api_submit.png)

##### 3\. 操作成功后，使用tar -xvf output.tar命令对任务结果进行解压

查看结果，可以看到解压后生成Stdout，Stderr,
tmp/result三个文件，打印其中的内容，分别为用户镜像中的标准输出，错误输出（也即Cmd参数），文件输出（也即POST内容）

如果访问失败，output中应为Json格式的错误信息。

![](/images/ugc_api_stdout.png)

### 通过API提交异步任务

这里演示通过shell命令演示通过API提交异步任务的过程。

##### 1\. 通过GetAccessToken API获取Token

注意填写正确的PublicKey和签名，签名算法见API文档概述。

![](/images/ugc_get_token.png)

##### 2\. 通过SubmitTask API提交任务

使用POST提交内容为“this is post data”

Content-Type设置为application/octet-stream

ImageName参数为用户提交的镜像名全称

TaskName为用户自定义任务名称，便于标记任务

AccessToken为上一步申请的Token参数

TaskType参数为Async表示异步任务

Cmd参数为运行镜像的输入参数，这里自定义为”thisiscmd”

OutputDir和OutputFileName参数为要输出的镜像文件内容

操作成功后返回任务的TaskId

![](/images/ugc_api_taskid.png)

通过GetTaskResult API获取任务的运行结果

![](/images/ugc_api_task_result.png)

##### 3\. 操作成功后，使用tar -xvf output.tar命令对任务结果进行解压

查看结果，可以看到解压后生成Stdout，Stderr,
tmp/result三个文件，打印其中的内容，分别为用户镜像中的标准输出，错误输出（也即Cmd参数），文件输出（也即POST内容）

如果访问失败，output中应为Json格式的错误信息。

![](/images/ugc_api_stdout.png)
