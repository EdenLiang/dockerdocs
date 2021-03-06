# 命令行（二）

## attach
	
	Usage: docker attach [OPTIONS] CONTAINER
	
	Attach to a running container  --no-stdin=false
	
	Do not attach STDIN  --sig-proxy=true
	
	Proxy all received signals to the process (non-TTY mode only). SIGCHLD, SIGKILL,and SIGSTOP are not proxied.

attach命令让你看到或者交互任何运行的容器的主进程（pid 1）。

你可以attach包含相同进程多次，屏幕共享风格，快速查看你守护进程的进度。

> 说明：这些命令不是针对正要运行的新进程，在容器里。请转到docker exec。

你可以从容器再次分离运行，使用CTRL-p CTRL-q停止，或者CTRL-c 发送SIGKILL给容器，或者CTRL-\得到堆栈跟踪docker client，当它退出时。当你分离容器进程退出代码是，将放回到docker client。

为了停止容器，使用docker stop。

为了kill掉容器，使用docker kill。

示例：

	$ ID=$(sudo docker run -d ubuntu /usr/bin/top -b)
	
	$ sudo docker attach
	
	$ID top
	
	-02:05:52 up  3:05,0 users,  load average:0.01,0.02,0.05
	
	Tasks:1 total,1 running,0 sleeping,0 stopped,0 zombie
	
	Cpu(s):0.1%us,0.2%sy,0.0%ni,99.7%id,0.0%wa,0.0%hi,0.0%si,0.0%st
	
	Mem:373572k total,355560k used,18012k free,27872k buffers
	
	Swap:786428k total,0k used,786428k free,221740k cached
	
	PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  COMMAND 1 root      200172001116
	
	912 R    00.30:00.03 top top -02:05:55 up  3:05,0 users,  load average:0.01,0.02,0.05Tasks:1 total,1 running,0 sleeping,0 stopped,0 zombie
	
	Cpu(s):0.0%us,0.2%sy,0.0%ni,99.8%id,0.0%wa,0.0%hi,0.0%si,0.0%st 
	
	Mem:373572k total,355244k used,18328k free,27872k buffers
	
	Swap:786428k total,0k used,786428k free,221776k cached   
	
	PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  COMMAND       1 root      200172081144
	
	932 R    00.30:00.03 top top -02:05:58 up  3:06,0 users,  load average:0.01,0.02,0.05
	
	Tasks:1 total,1 running,0 sleeping,0 stopped,0 zombie Cpu(s):0.2%us,0.3%sy,0.0%ni,99.5%id,0.0%wa,0.0%hi,0.0%si,0.0%st Mem:373572k total,355780k used,17792k free,27880k buffers
	
	Swap:786428k total,0k used,786428k free,221776k cached
	
	PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  COMMAND      1 root      200172081144932 R    00.30:00.03 top
	
	^C$
	
	$ sudo docker stop $ID

在第二个示例，你可以看到使用docker attach命令停止时通过bash进程返回的code。

	$ sudo docker run --name test -d -it debian
	275c44472aebd77c926d4527885bb09f2f6db21d878c75f0a1c212c03d3bcfab
	$ sudo docker attach test
	$$ exit 13
	exit
	$ echo $?
	13
	$ sudo docker ps -a | grep test
	275c44472aeb        debian:7            "/bin/bash"         26 seconds ago      Exited (13) 17 seconds ago                         test


## build
	
	 
	
	Usage: docker build [OPTIONS] PATH | URL |-
	
	Build a new image from the source code at PATH  --force-rm=false
	
	Always remove intermediate containers, even after unsuccessful builds  --no-cache=false
	
	Do not use cache when building the image  --pull=falseAlways attempt to pull a newer version of the image  -q,--quiet=false
	
	Suppress the verbose output generated by the containers  --rm=true
	
	Remove intermediate containers after a successful build  -t,--tag=""Repository name (and optionally a tag) to be applied to the resulting image incase of success

使用这个命令构建docker镜像，从dockerfile和"上下文"。

文件在PATH或者URL被成为构建的"上下文"。构建进程在上下文的文件中提供任何内容，例如，当使用一个ADD指令。当一个单独的dockerfile被给定URL或者通过指定STDIN（`docker build - < Dockerfile`），然后没有上下文被设置。

当一个Git库被设置为URL，其库被使用为上下文。Git库使用它的子模块（git clone -recursive）。一个刷新的git clone发生在一个临时文件在你的本地主机里。然后被发送到docker daemon作为上下文。这样，你的本地用户证书和VPN等等被使用来验证私有证书。

如果一个文件命名为.dockerignore存在在PATH的根目录，然后它将被解析作为一个换行符分开的exclusion patterns列表。exclusion patterns匹配文件或者目录，相对于PATH将被排除在上下文。通配符被是用来作为Go语言的filepath.Match规则。

请说明.dockerignore 文件在其它子目录被考虑为一个普通文件。文件路径在.dockerignore是相对于当前路径做为根目录是绝对的。通配符被允许，当时search不是回归的。

示例， .dockerignore 文件：

*/temp*

*/*/temp*

temp?

第一行*/temp*将忽略所有文件带有起始名字为temp 任何子目录，在根目录下。例如，一个文件叫做/somedir/temporary.txt 将被忽略。第二行*/*/temp*，将忽略文件以命名temp的所有第二级子目录，在根目录下。例如：/somedir/subdir/temporary.txt将被忽略。最后行temp?将忽略文件根目录下的部分文件。当前这些不被支持的规则表达式，例如格式[^temp*] 被忽略。

这里可以去查看：

Dockerfile Reference.

示例：

	$ sudo docker build .
	
	Uploading context 10240 bytes
	
	Step1: FROM busybox
	
	Pulling repository busybox ---> e9aa60c60128MB/2.284 MB (100%) endpoint: https://cdn-registry-1.docker.io/v1/
	
	Step2: RUN ls -lh /
	
	--->Runningin9c9e81692ae9total 24drwxr-xr-x    2 root     root        4.0KMar122013 bindrwxr-xr-x    5 root     root        4.0KOct1900:19 devdrwxr-xr-x    2 root     root        4.0KOct1900:19 etcdrwxr-xr-x    2 root     root        4.0KNov1523:34 liblrwxrwxrwx    1 root     root           3Mar122013 lib64 -> libdr-xr-xr-x  116 root     root           0Nov1523:34 proclrwxrwxrwx    1 root     root           3Mar122013 sbin -> bindr-xr-xr-x   13 root     root           0Nov1523:34 sysdrwxr-xr-x    2 root     root        4.0KMar122013 tmpdrwxr-xr-x    2 root     root        4.0KNov1523:34 usr ---> b35f4035db3f
	
	Step3: CMD echo Hello world
	
	--->Runningin02071fceb21b---> f52f38b7823eSuccessfully built f52f38b7823eRemoving intermediate container 9c9e81692ae9Removing intermediate container 02071fceb21b

示例指定PATH 为.。所有文件都在本地目录下，获取tar和发送给docker daemon。PATH指定文件哪里被发现并被作为上下文。记住，docker daemon运行在远程的机器上，这不会解析dockerfile发生在客户端.这意味着，所有文件都会被发送，不仅仅是列举到ADD在dockerfile里。

上下文转移到本地机器，docker client意味着当你看到“Sending build context”后。

如果你希望保持中间容器，在构建完成后，你必须使用--rm=false。这不会影响构建的缓存。

	$ sudo docker build .
	
	Uploading context 18.829 MB
	
	Uploading context
	
	Step0: FROM busybox
	
	--->769b9341d937
	
	Step1: CMD echo Hello world
	
	--->Using cache --->99cc1ad10469
	
	Successfully built 99cc1ad10469$ echo ".git">.dockerignore
	
	$ sudo docker build .
	
	Uploading context  6.76 MB
	
	Uploading context
	
	Step0: FROM busybox --->769b9341d937
	
	Step1: CMD echo Hello world
	
	--->Using cache --->99cc1ad10469
	
	Successfully built 99cc1ad10469

这个例子显示了.dockerignore 的使用，以此来忽略.git目录。它影响可以通过大小的改变来看到。

	$ sudo docker build -t vieux/apache:2.0.

这将构建先前的示例，但是它将tag 结果镜像。仓库名称将是vieux/apache，标签将是2.0。

	$ sudo docker build -<Dockerfile

这将读取一个dockerfile，来自STDIN。如果缺少上下文，任何本地文件都没有内容，也将被发送给Docker daemon。自从这些没有上下文，dockerfile的ADD仅工作，如果它指定一个远程URL。

	$ sudo docker build -< context.tar.gz

这将构建一个镜像，为压缩上线问阅读来自STDIN。支持格式是bzip2,gzip和xz。

	$ sudo docker build github.com/creack/docker-firefox

这将clone Github库，使用复制的库作为上下文。dockerfile在根目录下的库被作为dockerfile。说明，你可以指定一个任意的Git库，通过使用git:// 或者 git@ 概要。

> 说明：docker build将返回一个“no such file or directory”错误，如果文件或者目录不存在在上传的上下文中。这可能会发生，如果没有上下文，或者如果你指定一个文件，它在主机系统的别个地方。上下文被限制在当前目录或者它的子目录下作为安全原因，为了确保可重复构建在远程的Docker host。这也是一个理由，为什么ADD ../file将不起作用。

## commit

	Usage: docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]
	
	Create a new image from a container's changes  -a, --author=""     
	
	Author (e.g., "John Hannibal Smith <hannibal@a-team.com>")  -m, --message=""    
	
	Commit message  -p, --pause=true    
	
	Pause container during commit

它是有用的commit 容器文件的修改或者设置进新的镜像。这运行你调试容器通过运行一个交互的shell，或者export一个可工作的dataset到另一个服务。普遍地，它是最好的，使用dockerfile管理你的镜像在一个文档或者维护的方法。

默认情况下，容器将被commit并且它的进程将被停止，当镜像被commit时。这减少了创建提交过程中遇到的数据损坏的可能性。如果这种行为不被期望，设置p参数以此来关闭。

1. Commit 一个已经存在的container

	$ sudo docker ps
	
	ID                  IMAGE               COMMAND             CREATED             STATUS              PORTSc3f279d17e0a        ubuntu:12.04/bin/bash           7 days ago          
	
	Up25 hours197387f1b436        ubuntu:12.04/bin/bash           7 days ago          
	
	Up25 hours
	
	$ sudo docker commit c3f279d17e0a  
	
	SvenDowideit/testimage:version3f5283438590d
	
	$ sudo docker images | head
	
	REPOSITORY                        TAG                 ID                  CREATED             VIRTUAL SIZE
	
	SvenDowideit/testimage            version3            f5283438590d        16 seconds ago      335.7 MB


Commit一个容器使用新的配置项

	$ sudo docker ps
	ID                  IMAGE               COMMAND             CREATED             STATUS              PORTS
	c3f279d17e0a        ubuntu:12.04        /bin/bash           7 days ago          Up 25 hours
	197387f1b436        ubuntu:12.04        /bin/bash           7 days ago          Up 25 hours
	$ sudo docker inspect -f "{{ .Config.Env }}" c3f279d17e0a
	[HOME=/ PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin]
	$ sudo docker commit --change "ENV DEBUG true" c3f279d17e0a  SvenDowideit/testimage:version3
	f5283438590d
	$ sudo docker inspect -f "{{ .Config.Env }}" f5283438590d
	[HOME=/ PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin DEBUG=true]

###　cp

复制文件/文件夹，从容器文件系统到主机路径下。路径是相对的文件系统的根目录的。

	Usage: docker cp CONTAINER:PATH HOSTPATH
	
	Copy files/folders from the PATH to the HOSTPATH

## create

创建一个新的容器

	Usage: docker create [OPTIONS] IMAGE [COMMAND][ARG...]
	
	Create a new container  -a,--attach=[]
	
	Attach to STDIN, STDOUT or STDERR.--add-host=[]
	
	Add a custom host-to-IP mapping (host:ip)-c,--cpu-shares=0         CPU shares (relative weight)--cap-add=[]
	
	Add Linux capabilities  --cap-drop=[]
	
	Drop Linux capabilities  --cidfile=""
	
	Write the container ID to the file  --cpuset=""
	
	CPUs in which to allow execution (0-3,0,1)--device=[]
	
	Add a host device to the container (e.g.--device=/dev/sdc:/dev/xvdc:rwm)--dns=[]
	
	Set custom DNS servers  --dns-search=[]
	
	Set custom DNS search domains (Use--dns-search=.if you don't wish to set the search domain)  -e, --env=[]               
	
	Set environment variables  --entrypoint=""            
	
	Overwrite the default ENTRYPOINT of the image  --env-file=[]              
	
	Read in a line delimited file of environment variables  --expose=[]                
	
	Expose a port or a range of ports (e.g. --expose=3300-3310) from the container without publishing it to your host  -h, --hostname=""          Container host name  -i, --interactive=false    Keep STDIN open even if not attached  --ipc=""                   
	
	Default is to create a private IPC namespace (POSIX SysV IPC) for the container                               'container:<name|id>': reuses another container shared memory, semaphores and message queues                               'host': use the host shared memory,semaphores and message queues inside the container.  
	
	Note: the host mode gives the container full access to local shared memory and is therefore considered insecure.  --link=[]                  
	
	Add link to another container in the form of name:alias  --lxc-conf=[]              (lxc exec-driver only) 
	
	Add custom lxc options --lxc-conf="lxc.cgroup.cpuset.cpus = 0,1"  -m, --memory=""            
	
	Memory limit (format: <number><optional unit>, where unit = b, k, m or g)  --mac-address=""           
	
	Container MAC address (e.g. 92:d0:c6:0a:29:33)  --name=""                  
	
	Assign a name to the container  --net="bridge"             
	
	Set the Network mode for the container                               'bridge': creates a new network stack for the container on the docker bridge                               'none': no networking for this container                               'container:<name|id>': reuses another container network stack                               'host': use the host network stack inside the container. 
	
	Note: the host mode gives the container full access to local system services such as D-bus and is therefore considered insecure.  -P, --publish-all=false    Publish all exposed ports to the host interfaces  -p, --publish=[]           
	
	Publish a container's port to the host                               format: ip:hostPort:containerPort | ip::containerPort | hostPort:containerPort | containerPort                               (use'docker port' to see the actual mapping)--privileged=falseGive extended privileges to this container  --restart=""
	
	Restart policy to apply when a container exits (no, on-failure[:max-retry], always)--security-opt=[]
	
	SecurityOptions-t,--tty=falseAllocate a pseudo-TTY  -u,--user=""Usernameor UID  -v,--volume=[]
	
	Bind mount a volume (e.g.,from the host:-v /host:/container, from Docker: -v /container)--volumes-from=[]
	
	Mount volumes from the specified container(s)-w,--workdir=""
	
	Working directory inside the container

docker create命令在指定镜像基础上创建一个可写容器层，其指令包含正在运行的指定命令。容器ID是稍后通过STDOUT输出的。这类似与docker run -d接收容器不是被启动的。你可以稍后使用`docker start <container_id>`命令启动容器。

这是有用的，当你想安装一个容器配置，在准备启动之前。

> 说明，volumes 通过create重写参数设置，通过start命令。

请查看"run command"节了解更多详细内容。

例如：

	$ sudo docker create -t -i fedora bash
	
	6d8af538ec541dd581ebc2a24153a28329acb5268abe5ef868c1f1a261221752
	
	$ sudo docker start -a -i 6d8af538ec5
	
	bash-4.2#

v1.4.0容器数据卷被初始化，在docker create阶段（或者docker run阶段）。例如，这允许你创建数据卷的容器，然后使用来自于其他容器。

	$ docker create -v /data --name data ubuntu
	
	240633dfbb98128fa77473d3d9018f6123b99c454b3251427ae190a7d951ad57
	
	$ docker run --rm --volumes-from data ubuntu ls -la /data
	
	total 8
	
	drwxr-xr-x  2 root root 4096Dec504:10.
	
	drwxr-xr-x 48 root root 4096Dec504:11..

相似地，create 一个主机目录绑定到数据卷容器，然后被随后创建的容器使用。

	$ docker create -v /home/docker:/docker --name docker ubuntu
	
	9aa88c08f319cd1e4515c3c46b0de7cc9aa75e878357b1e96f91e2c773029f03
	
	$ docker run --rm --volumes-from docker ubuntu ls -la /docker
	
	total 20
	
	drwxr-sr-x  51000 staff  180Dec504:00.
	
	drwxr-xr-x 48 root root  4096Dec504:13..
	
	-rw-rw-r--11000 staff 3833Dec504:01.ash_history
	
	-rw-r--r--11000 staff  446Nov2811:51.ashrc
	
	-rw-r--r--11000 staff   25Dec504:00.gitconfig
	
	drwxr-sr-x  31000 staff   60Dec103:28.local
	
	-rw-r--r--11000 staff  920Nov2811:51.profile
	
	drwx--S---21000 staff  460Dec500:51.ssh
	
	drwxr-xr-x 321000 staff 1140Dec504:01 docker


## diff

列举容器文件系统，文件和目录所做的修改。
	
	Usage: docker diff CONTAINER
	
	Inspect changes on a container's filesystem

这里有3个事件，在diff命令中列举出来。

	A - Add
	
	D - Delete
	
	C - Change

例如：

	$ sudo docker diff 7bb0e258aefe
	
	C /dev
	
	A /dev/kmsg
	
	C /etc
	
	A /etc/mtab
	
	A /go
	
	A /go/src
	
	A /go/src/github.com
	
	A /go/src/github.com/docker
	
	A /go/src/github.com/docker/docker
	
	A /go/src/github.com/docker/docker/.git....

## events

 

	Usage: docker events [OPTIONS]
	
	Get real time events from the server  
	
	    -f,--filter=[]Provide filter values (i.e.,'event=stop')
	
	    --since=""Show all events created since timestamp  
	
	    --until=""Stream events untilthis timestamp

Docker 容器将报告如下事件：

	create, destroy,die,export, kill, pause, restart, start, stop, unpause

Docker镜像将会报告如下事件：

	untag,delete

1、过滤

过滤标志(-f 或者 --filter)格式是"key=value"的形式。如果你将使用多个过滤，可以指定多个标志(例如，--filter "foo=bar" --filter "bif=baz")。

使用相同的filter多次将被处理多次OR，例如--filter container=588a23dac085 --filter container=a8f7720b8c22将被视为查询容器588a23dac085或者容器a8f7720b8c22的事件。

当前能过滤的有* event * image * container。

示例：

你将需要两个shell，在这个例子中。

	Shell 1: Listening for events:
	
	$ sudo docker events
	
	Shell 2: Start and Stop containers:
	
	$ sudo docker start 4386fb97867d
	
	$ sudo docker stop 4386fb97867d
	
	$ sudo docker stop 7805c1d35632
	
	Shell 1: (Again .. now showing events):
	
	2014-05-10T17:42:14.999999999Z07:004386fb97867d:(from ubuntu-1:14.04) start
	
	2014-05-10T17:42:14.999999999Z07:004386fb97867d:(from ubuntu-1:14.04)die
	
	2014-05-10T17:42:14.999999999Z07:004386fb97867d:(from ubuntu-1:14.04) stop
	
	2014-05-10T17:42:14.999999999Z07:007805c1d35632:(from redis:2.8)die
	
	2014-05-10T17:42:14.999999999Z07:007805c1d35632:(from redis:2.8) stop
	
	Show events in the past from a specified time:
	
	$ sudo docker events --since 1378216169
	
	2014-03-10T17:42:14.999999999Z07:004386fb97867d:(from ubuntu-1:14.04)die
	
	2014-05-10T17:42:14.999999999Z07:004386fb97867d:(from ubuntu-1:14.04) stop
	
	2014-05-10T17:42:14.999999999Z07:007805c1d35632:(from redis:2.8)die
	
	2014-03-10T17:42:14.999999999Z07:007805c1d35632:(from redis:2.8) stop
	
	$ sudo docker events --since '2013-09-03'
	
	2014-09-03T17:42:14.999999999Z07:004386fb97867d:(from ubuntu-1:14.04) start
	
	2014-09-03T17:42:14.999999999Z07:004386fb97867d:(from ubuntu-1:14.04)die
	
	2014-05-10T17:42:14.999999999Z07:004386fb97867d:(from ubuntu-1:14.04) stop
	
	2014-05-10T17:42:14.999999999Z07:007805c1d35632:(from redis:2.8)die
	
	2014-09-03T17:42:14.999999999Z07:007805c1d35632:(from redis:2.8) stop
	
	$ sudo docker events --since '2013-09-03 15:49:29 +0200 CEST'
	
	2014-09-03T15:49:29.999999999Z07:004386fb97867d:(from ubuntu-1:14.04)die
	
	2014-05-10T17:42:14.999999999Z07:004386fb97867d:(from ubuntu-1:14.04) stop
	
	2014-05-10T17:42:14.999999999Z07:007805c1d35632:(from redis:2.8)die
	
	2014-09-03T15:49:29.999999999Z07:007805c1d35632:(from redis:2.8) stop
	
	Filter events:
	
	$ sudo docker events --filter 'event=stop'
	
	2014-05-10T17:42:14.999999999Z07:004386fb97867d:(from ubuntu-1:14.04) stop
	
	2014-09-03T17:42:14.999999999Z07:007805c1d35632:(from redis:2.8) stop
	
	$ sudo docker events --filter 'image=ubuntu-1:14.04'
	
	2014-05-10T17:42:14.999999999Z07:004386fb97867d:(from ubuntu-1:14.04) start
	
	2014-05-10T17:42:14.999999999Z07:004386fb97867d:(from ubuntu-1:14.04)die
	
	2014-05-10T17:42:14.999999999Z07:004386fb97867d:(from ubuntu-1:14.04) stop
	
	$ sudo docker events --filter 'container=7805c1d35632'
	
	2014-05-10T17:42:14.999999999Z07:007805c1d35632:(from redis:2.8)die
	
	2014-09-03T15:49:29.999999999Z07:007805c1d35632:(from redis:2.8) stop
	
	$ sudo docker events --filter 'container=7805c1d35632'--filter 'container=4386fb97867d'
	
	2014-09-03T15:49:29.999999999Z07:004386fb97867d:(from ubuntu-1:14.04)die
	
	2014-05-10T17:42:14.999999999Z07:004386fb97867d:(from ubuntu-1:14.04) stop
	
	2014-05-10T17:42:14.999999999Z07:007805c1d35632:(from redis:2.8)die
	
	2014-09-03T15:49:29.999999999Z07:007805c1d35632:(from redis:2.8) stop
	
	$ sudo docker events --filter 'container=7805c1d35632'--filter 'event=stop'
	
	2014-09-03T15:49:29.999999999Z07:007805c1d35632:(from redis:2.8) stop