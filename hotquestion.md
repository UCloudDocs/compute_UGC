

# 常见问题

### 内网访问UGC很慢

访问UGC地址api.ugc.service.ucloud.cn很慢，有可能是您修改了uhost的默认路由导致的，新增一条路由规则指向ugc服务即可。
\`\`\` ip route add 10.19.0.0/16 via 10.x.0.1 \`\`\`

### UGC任务容器实例是否有网络通信能力？比如从UFile里下载文件，上传文件，或者访问外网等等?

UGC的任务容器没有网络通信能力，不能访问UFile服务或者外网。UGC任务作为请求服务的终端，不再向外发送网络请求；UGC只专注计算任务。建议您在调用UGC之前将要处理的原始数据从UFile里先下载下来，然后将原始数据POST提交到UGC的提交任务API，然后UGC的API返回计算结果数据。

### 提交了任务后，能否登录到容器里实时查看任务程序的运行情况？

作为Serverless产品，UGC为您的计算任务自动调度分配计算资源创建实例，但您不能登录您的计算实例容器。

### API SubmitTask里的OutputDir和OutputFileName参数是什么意思？有什么作用？

用户可以通过这两个参数指定容器镜像内部某个目录文件作为计算结果的文件数据，然后在算法代码中将需要返回的计算结果数据写入该目录文件中。UGC任务执行成功后，同步任务可以通过API
SubmitTask的响应报文里拿到包含stdout, stderr和该目录文件的Tar包文件集；异步任务可以通过调用API
GetTaskResult获取相应的Tar包文件集。

### 那如何指定多个文件作为计算任务返回的结果文件集？

您可以在API SubmitTask中指定多个OutputFileName参数，例如

<http://api.ugc.service.ucloud.cn/?Action=SubmitTask&OutputDir=%2ftmp&OutputFileName=Afile&OutputFileName=Bfile>

### 为什么UGC的任务代码必须做无状态改造？

UGC能为您迅速扩容出足够多的任务实例来应对您的并发请求，但您无法预料具体某个任务运行于哪个实例中，任务代码无状态化能够使您无需关注任务实例的分配详情。

### 我在本地开发机上调用UGC的API SubmitTask，感觉调用延迟非常高？

建议您将调用API
SubmitTask的程序从您的本地开发机迁移到UHost云主机中。本地开发机需要通过公网访问api.ugc.service.ucloud.cn域名，可能会由于公网带宽小，传输不稳定导致接口耗时较大；而在UHost云主机则是通过UCloud内网访问api.ugc.service.ucloud.cn域名，带宽高且传输稳定，网络延迟低。

### 向UGC私有镜像仓库成功的推送了算法任务镜像后，多长时间后UGC平台能运行我的算法任务实例？

镜像部署时间取决于您的镜像大小。一般而言，UGC的算法任务镜像需要几十秒到数分钟即可完成部署以供调用。

### UGC任务容器内是否适合运行服务端程序？

不适合。原因：1）UGC任务容器没有网络通信能力，无法为客户端提供网络服务；2）服务端程序通常为永不退出的常驻进程，然而无论是同步任务还是异步任务，UGC都有对应任务容器最大运行时间（同步任务为300秒，异步任务为24小时），超过此时间后任务容器被系统自动终止。
