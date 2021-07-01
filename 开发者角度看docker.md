在谈开发者角度理解docker之前，我们先来了解下docker - 这个通过容器技术解决虚拟机模式的缺点的跨时代产物。

下面我们来看一下docker的架构以及它容器模式相对于虚拟机的优势

#### docker架构
* 镜像（images）- 由多个镜像层组合成（Dockerfile每一条指令生成一层镜像层，可通过&连接符连续执行指令生成一层镜像层，Dockerfile文件复杂的情况下也可以通过多Dockerfile文件多阶段构建镜像，避免形成臃肿、层级过多的镜像），包含精简的操作系统和应用运行的必须的文件和依赖包。
* 容器（container）- 镜像的实例，将可运行的应用打包到轻量级的容器上（应用即容器），能发布到任何支持容器运行时环境linux机器上（可移植）。
* 仓库 - 存放所有镜像的仓库。

#### docker容器模式的优势
* 轻量级 - 运行在同一宿主机上的容器共享操作系统，且将操作系统虚拟化，而虚拟机是每一个虚拟机（应用）都有自己专用的操作系统,操作系统通过物理硬件分配虚拟操作系统,每个虚拟机都是包含了虚拟CPU、虚拟RAM、虚拟磁盘等系统资源。
* 启动快 - 直接使用宿主机的系统内核，避免了虚拟机启动时所需要的系统引导时间和操作系统运行的资源消耗,且容器使用的资源远远少于虚拟机。虚拟机需要通过Hypervisor实现硬件资源的虚拟化,每个虚拟机都通过操作系统声明、初始化、管理这些虚拟资源造成很多额外的开销。
* 可移植 - docker引擎可以获取共享操作系统的系统资源，在容器内部抽象出一个操作系统（虚拟化），在内部就可以运行应用程序。所以容器可以移植到任何支持容器运行时环境linux机器上。

这个比较docker和虚拟机并没有捧高踩低的意思，每种新技术的诞生都是历史发展必然的结果，它们都有着自己独特的优点，在某些使用场景中，虚拟机模式优于容器模式，例如虚拟机更擅长于彻底隔离整个运行环境。

从开发者角度看docker,我们只需要关注实现应用容器化的过程。通过git操作拉取项目代码，打包项目代码以及安装依赖，把能够运行的项目代码以及其运行需要的文件和依赖包通过执行Dockerfile的指令构建到镜像层上，运行容器实现应用容器化。

#### Dockerfile指令详解
* FROM - 指定基础镜像，其他的Dockerfile指令构建的镜像层都在这个之上。
* COPY - 将<源路径>的文件复制到<目标路径>上。
* ADD - 功能几乎和COPY一样，增加来文件自动解压的功能，如果 <源路径> 为一个 tar 压缩文件的话，压缩格式为 gzip, bzip2 以及 xz 的情况下，ADD 指令将会自动解压缩这个压缩文件到 <目标路径> 去。
* EXPOSE - 声明容器运行时提供服务的端口，但是在容器运行时不会因为这个声明就会开启这个端口的服务。但是使用随机端口映射时（docker run -P）会自动映射 EXPOSE 的端口上。
* HEALTHCHECK - 健康检查指令，告诉 Docker 应该如何进行判断（--interval=<间隔>: 两次健康检查的间隔，--timeout=<时长>: 健康检查命令运行超时时间，如果超过这个时间，本次健康检查就被视为失败，--retries=<次数>: 当连续失败指定次数后，则将容器状态视为 unhealthy）容器的状态是否正常。
* CMD - 容器启动指令。
* RUN - 执行命令行命令。

#### bosslist项目实现应用容器化
bosslist使用shell脚本完成一键化部署，通过shell脚本打包项目代码、安装依赖，执行Dockerfile文件的指令构建镜像，打tag（如果不声明tag名称，会默认打上：latest tag。也会出现悬虚镜像的情况（无tag镜像），就是构建新镜像时打一个已经存在的镜像，docker会移除存在相同tag镜像的tag,为新镜像打上tag，旧tag镜像就变成悬虚镜像），发布镜像。k8s完成容器的更新以及容器的运行。

```javascript
echo "开始编译!"
cd ../.. && yarn && yarn build:prod

cp -r package.json yarn.lock bin dist ./docker/temp/${project_name}

cp WW_verify_bryxATIWmGJNo7fZ.txt ./docker/temp/${project_name}/dist

cd docker/temp/${project_name}  && yarn install --production 

cd .. && tar czf ${project_name}.tar.gz ${project_name} && rm -rf ${project_name}

echo "开始build镜像!"
# docker build -f ../Dockerfile -t ${local_image} .

if [ $1 = "prod" ]; then
    docker build -f ../Dockerfile -t ${local_image} .

echo "开始打镜像"
docker tag ${local_image} ${remote_store}:${commit_short_id}

echo "开始登录docker 对应的仓库地址"
docker login --username=${user} --password=${password} ${remote_store}

echo "开始发布相应的镜像"
docker push ${remote_store}:${commit_short_id}

echo commit_short_id

echo "更新容器"
kubectl --kubeconfig ~/.kube/${kubeconfig} set image Deployment/app-bosslist-front app-bosslist-front=${remote_store}:${commit_short_id}
```

在bosslist项目部署中，还使用nodejs（express框架）做了中间层处理。主要的处理是接口的反向代理和一些特殊操作。

### 参考配置
1. 深入浅出Docker