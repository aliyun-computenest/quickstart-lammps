# LAMMPS Ehpc计算巢快速部署


>**免责声明：**本服务由第三方提供，我们尽力确保其安全性、准确性和可靠性，但无法保证其完全免于故障、中断、错误或攻击。因此，本公司在此声明：对于本服务的内容、准确性、完整性、可靠性、适用性以及及时性不作任何陈述、保证或承诺，不对您使用本服务所产生的任何直接或间接的损失或损害承担任何责任；对于您通过本服务访问的第三方网站、应用程序、产品和服务，不对其内容、准确性、完整性、可靠性、适用性以及及时性承担任何责任，您应自行承担使用后果产生的风险和责任；对于因您使用本服务而产生的任何损失、损害，包括但不限于直接损失、间接损失、利润损失、商誉损失、数据损失或其他经济损失，不承担任何责任，即使本公司事先已被告知可能存在此类损失或损害的可能性；我们保留不时修改本声明的权利，因此请您在使用本服务前定期检查本声明。如果您对本声明或本服务存在任何问题或疑问，请联系我们。

## 概述


LAMMPS（Large-scale Atomic/Molecular Massively Parallel Simulator）是一款经典分子动力学软件。 LAMMPS包含的势函数可用于固体材料（金属、半导体）、软物质（生物大分子，聚合物）、粗粒化或介观尺度模型体系。
## 前提条件

部署LAMMPS社区版服务实例，需要对部分阿里云资源进行访问和创建操作。因此您的账号需要包含如下资源的权限。
  **说明**：当您的账号是RAM账号时，才需要添加此权限。

| 权限策略名称                          | 备注                     |
|---------------------------------|------------------------|
| AliyunECSFullAccess             | 管理云服务器服务（ECS）的权限       |
| AliyunVPCFullAccess             | 管理专有网络（VPC）的权限         |
| AliyunROSFullAccess             | 管理资源编排服务（ROS）的权限       |
| AliyunEHPCFullAccess             | 管理弹性高性能计算（EHPC）的权限       |
| AliyunNASFullAccess             | 管理文件存储（NAS）的权限       |
| AliyunComputeNestUserFullAccess | 管理计算巢服务（ComputeNest）的用户侧权限 |


## 计费说明

LAMMPS社区版在计算巢部署的费用主要涉及：

- 弹性高性能计算集群（EHPC）费用
- 文件系统（NAS）费用
- 流量带宽费用

## 部署架构
<img src="1.png" width="1500" height="700" align="bottom"/>

- 部署由一个ehpc集群组成，ehpc集群中包括manager节点、schedule节点和compute节点
- 服务使用nas-cpfs构建高性能共享文件系统

## 参数说明
| 参数组       | 参数项      | 说明                                                                     |
|-----------|----------|------------------------------------------------------------------------|
| 服务实例      | 服务实例名称   | 长度不超过64个字符，必须以英文字母开头，可包含数字、英文字母、短划线（-）和下划线（_）                          |
|           | 地域       | 服务实例部署的地域                                                              |
|           | 付费类型     | 资源的计费类型：按量付费和包年包月                                                      |
| EHPC集群配置  | 集群登录密码   | 长度8-30，必须包含三项（大写字母、小写字母、数字、 ()`~!@#$%^&*-+=&#124;{}[]:;'<>,.?/ 中的特殊符号） | 
|           | Ehpc部署模式 | Tiny，Simple，Standard                                                   |
|           | 计算节点实例类型 | 可用区下可以使用的计算节点规格                                                        |
|           | 计算节点数量   | 计算节点数量, 可选值: 1-99                                                      |
|           | 登录节点实例类型 | 可用区下可以使用的登录节点规格                                                        |
|           | 管控节点数量   | 管控节点数量, 可选值: 1,2,4                                                     |
| EHPC集群用户配置 | 用户密码     | 长度8-30，必须包含三项（大写字母、小写字母、数字、()~!@#$%^&*-_+=\                            |{}[]:;'/<>,.?/中的特殊符号） |
|                | 用户名称     | 登录集群所用的用户名，默认为lammps                                                   |
| 网络配置      | 可用区      | ECS实例所在可用区                                                             |
|           | VPC ID   | 资源所在VPC                                                                |
|           | 交换机ID    | 资源所在交换机                                                                |

## 部署流程
1. 访问计算巢LAMMPS社区版[部署链接](https://computenest.console.aliyun.com/user/cn-hangzhou/serviceInstanceCreate?&ServiceId=service-199f5aeaf0f142918076)
，按提示填写部署参数：
    ![image.png](2.png)

2. 参数填写完成后可以看到对应询价明细，确认参数后点击**下一步：确认订单**。
   ![image.png](3.png)

3. 确认订单完成后同意服务协议并点击**立即创建**
   进入部署阶段。
    ![image.png](4.png)

## 使用流程
### 步骤一：通过控制台连接集群
1. 登录[弹性高性能计算控制台](https://ehpc.console.aliyun.com)。
2. 在顶部菜单栏左上角处，选择地域。
3. 在左侧导航栏，单击**集群**。
4. 在**集群**页面，找到在计算巢部署好的目标集群，单击**远程连接**。
    <img src="5.png" height="700" align="bottom"/>

5. 在**远程连接**页面，输入集群用户名、登录密码和端口，单击**ssh连接**。

### **步骤二：提交作业**

   本文介绍如何使用E-HPC集群运行LAMMPS开源仿真软件，以3d Lennard-Jones melt模型进行工业仿真，并通过可视化的方式查看仿真结果。

1. 执行以下命令创建算例文件，算例文件命名为lj.in。

   ```
   vim lj.in
   ```
      
   作业脚本内容示例如下：

   \# 3d Lennard-Jones melt

   variable        x index 1

   variable        y index 1

   variable        z index 1

   variable        xx equal 20*$x

   variable        yy equal 20*$y

   variable        zz equal 20*$z

   units           lj

   atom_style      atomic

   lattice         fcc 0.8442

   region          box block 0 ${xx} 0 ${yy} 0 ${zz}

   create_box      1 box

   create_atoms    1 box

   mass            1 1.0

   velocity        all create 1.44 87287 loop geom

   pair_style      lj/cut 2.5

   pair_coeff      1 1 1.0 1.0 2.5

   neighbor        0.3 bin

   neigh_modify    delay 0 every 20 check no

   fix             1 all nve

   dump 1 all xyz 100 sample.xyz

   run             10000

2.执行以下命令创建作业脚本文件，脚本文件命名为lammps.pbs。

    ```
    vim lammps.pbs
    ```
    
    >以下示例使用1个计算节点的32 vCPU，使用32个MPI任务进行高性能计算。请根据实际计算节点规格配置vCPU数，算力要求vCPU≥32。
    
    作业脚本内容示例如下：

    \#!/bin/sh

    \#PBS -l select=1:ncpus=32:mpiprocs=32

    \#PBS -j oe

    export MODULEPATH=/opt/ehpcmodulefiles/   #module命令依赖的环境变量

    module load lammps-openmpi/31Mar17

    module load openmpi/1.10.7

    echo "run at the beginning"

    mpirun lmp -in ./lj.in                    #请根据实际修改lj.in文件的路径

3.执行以下命令提交作业。

    ```
    qsub lammps.pbs
    ```

    预期返回如下，表示生成的作业ID为0.scheduler。

    ```
    0.scheduler
    ```

### **步骤三：查看作业结果**

1. 查看作业运行情况。

    ```
    cat lammps.pbs.o0
    ```
    
    >如果您不指定作业标准输出路径，则默认按照调度器行为生成输出文件。默认作业结果文件输出/home/<用户名>/目录下，本示例中的作业结果文件为/home/testuser/lammps.pbs.o0。

    预期返回如下:

    ......

    Per MPI rank memory allocation (min/avg/max) = 3.777 | 3.801 | 3.818 Mbytes

    Step Temp E_pair E_mol TotEng Press 

    0         1.44   -6.7733681            0   -4.6134356   -5.0197073 

    10000   0.69814375   -5.6683212            0   -4.6211383   0.75227555

    Loop time of 9.81493 on 32 procs for 10000 steps with 32000 atoms
    
    Performance: 440145.641 tau/day, 1018.856 timesteps/s

    97.0% CPU use with 32 MPI tasks x no OpenMP threads
    
    MPI task timing breakdown:

    |Section |  min time  |  avg time  |  max time  |%varavg| %total

    |---------------------------------------------------------------

    |Pair    | 6.0055     | 6.1975     | 6.3645     |   4.0 | 63.14

    |Neigh   | 0.90095    | 0.91322    | 0.92938    |   0.9 |  9.30

    |Comm    | 2.1457     | 2.3105     | 2.4945     |   6.9 | 23.54

    |Output  | 0.16934    | 0.1998     | 0.23357    |   4.3 |  2.04

    |Modify  | 0.1259     | 0.13028    | 0.13602    |   0.8 |  1.33

    |Other   |            | 0.06364    |            |       |  0.65

    Nlocal:    1000 ave 1022 max 986 min

    Histogram: 5 3 6 3 4 4 2 2 1 2

    Nghost:    2705.62 ave 2733 max 2668 min

    Histogram: 1 1 0 3 7 5 4 5 4 2

    Neighs:    37505 ave 38906 max 36560 min

    Histogram: 7 3 2 4 5 2 3 3 2 1
    
    Total # of neighbors = 1200161

    Ave neighs/atom = 37.505

    Neighbor list builds = 500

    Dangerous builds not checked

    Total wall time: 0:00:10
    
2. 使用VNC可视化查看作业结果。
    1. 打开VNC。 控制台操作时系统会自动打开集群安全组 12016 端口。 
       1. 在[弹性高性能计算控制台](https://ehpc.console.aliyun.com)的左侧导航栏，单击**集群**。 
       2. 在**集群**页面，找到目标集群，单击**更多** &gt; **VNC**。
       3. 使用VNC远程连接可视化服务。具体操作，请参见[连接可视化服务](https://help.aliyun.com/zh/e-hpc/user-guide/use-vnc-to-manage-a-visualization-service#section-bf6-eyn-edu)。
    2. 在VNC窗口中，选择**Application&gt;System Tools&gt;Terminal**。
    3. 运行`/opt/vmd/1.9.3/vmd`打开VMD软件。
    4. 在VMD Main对话框中，选择**File &gt; New Molecule...**。
    5. 单击**Filename**处对应的**Browse...**，选择结果文件sample.xyz。
    6. sample.xyz文件所在路径为/home/{用户名}/sample.xyz。
    7. 单击**Load**，可在**VMD 1.9.3 OpenGL Display**窗口查看可视化结果。
    
       <img src="7.png" height="800" align="bottom"/>