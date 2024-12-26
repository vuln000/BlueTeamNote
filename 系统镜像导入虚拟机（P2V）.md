<font style="color:rgba(0, 0, 0, 0.9);background-color:rgb(244, 251, 247);">该篇收录自 https://mp.weixin.qq.com/s/lUn8_fb0W8EU-ruBnkz74w</font>

<font style="color:rgba(0, 0, 0, 0.9);background-color:rgb(244, 251, 247);">vCenter Converter Standalone下载链接：https://blog.1e3.ru/vcenter-converter-standalone/</font>

# **第1章 前言**** **** **
VMware vCenter Converter Standalone是由VMware提供的工具，用于将物理机或其他第三方虚拟化产品的虚拟机转换成VMware虚拟机。它是一个免费且强大的工具，旨在简化将工作负载迁移至VMware vSphere的过程。vCenter Converter Standalone可以执行几种类型的转换，包括：

1.物理到虚拟（P2V）转换：将物理机转换为虚拟机。它允许将整个操作系统、应用程序和数据从物理服务器迁移到在VMware vSphere上运行的虚拟机。

2.虚拟到虚拟（V2V）转换：除了P2V转换外，vCenter Converter Standalone 还可以将其他虚拟化产品（如Microsoft Hyper-V或Citrix XenServer）上的虚拟机转换为VMware虚拟机。这对于将虚拟化基础架构整合到VMware 平台上非常有用。

3.虚拟到物理（V2P）转换：将虚拟机转换回物理机。虽然这种用例较为少见，但当您需要将虚拟机备份还原到物理服务器或将工作负载从虚拟环境迁移到物理服务器时，它可以发挥作用。

vCenter Converter Standalone 提供了高级功能和自定义选项，以优化转换过程。一些主要特点包括：

1.克隆和分区调整：您可以从源计算机克隆磁盘，在转换过程中调整分区大小，并排除不必要的数据以减小虚拟机的大小。

2.同步和测试：您可以多次执行转换过程，确保目标虚拟机与源计算机保持同步。它还提供了测试模式，可以在源计算机上运行转换后的虚拟机，以便在执行迁移之前进行测试。

3.预约和自动化：vCenter Converter Standalone 允许您安排在特定时间执行转换，或定期运行转换。它还提供了命令行界面（CLI），可用于脚本编写和自动化操作。

4.多卷和多磁盘系统的转换：该工具支持转换具有多个卷或磁盘的系统，包括动态磁盘等复杂磁盘配置。

本文主要介绍VMwarevCenter Converter Standalone的安装以及P2V的具体操作步骤。

# **第2章 准备工具  **
1、ESXi底层、vCenter。

2、VMware vCenter Converter Standalone工具（下载地址：https://customerconnect.vmware.com/en/downloads/#all_products）

3、Windows系统和Linux系统。

4、一台计算机。

# **第3章 拓扑图  **
![](https://cdn.nlark.com/yuque/0/2024/webp/38747917/1723793819417-e8b43762-9ab8-4d3c-81bc-bd5134b29725.webp)

# **第4章安装vCenter Converter Standalone  **
1、双击打开安装程序。

![](https://cdn.nlark.com/yuque/0/2024/webp/38747917/1723793819551-4b0f7ada-7ccb-4b72-a5af-d68b0a5a4997.webp)

2、点击“Next”下一步。

![](https://cdn.nlark.com/yuque/0/2024/webp/38747917/1723793819867-1a07f951-fb18-4ec3-84c0-423f4a43e11e.webp)

3、继续下一步。

![](https://cdn.nlark.com/yuque/0/2024/webp/38747917/1723793820160-0280849b-0c65-4816-b225-e0038c6b4a60.webp)

4、同意许可协议。

![](https://cdn.nlark.com/yuque/0/2024/webp/38747917/1723793820169-5e8387f6-648d-4969-8e69-41782798b09b.webp)

5、根据实际情况更改安装位置，点击下一步。

![](https://cdn.nlark.com/yuque/0/2024/webp/38747917/1723793820137-abbe7dfb-6e82-4d85-bd8a-d7a3f2502697.webp)

6、选择“本地安装”，点击下一步。

![](https://cdn.nlark.com/yuque/0/2024/webp/38747917/1723793820132-50db1b3b-4566-4be4-8a20-b8508ff5fd4a.webp)

7、取消勾选“VMware客户体验计划”（因为是实验环境），点击下一步。

![](https://cdn.nlark.com/yuque/0/2024/webp/38747917/1723793820274-542cef83-134b-4c7a-b3c6-81390adacee7.webp)

8、点击“install”开始安装。

![](https://cdn.nlark.com/yuque/0/2024/webp/38747917/1723793820489-a9820be2-bed9-4e41-8192-17b39070e21e.webp)

9、点击“finish”安装完成并打开控制台。

![](https://cdn.nlark.com/yuque/0/2024/webp/38747917/1723793820505-d7a5633f-a068-4cba-a44d-8fec7c776c19.webp)

10、控制台界面

![](https://cdn.nlark.com/yuque/0/2024/webp/38747917/1723793820626-be565a44-0eb7-4d53-9f35-9bd638636177.webp)

# **第5章对Windows系统设备进行P2V操作**** **** **
1、点击“Converter machine”进行P2V操作。

![](https://cdn.nlark.com/yuque/0/2024/webp/38747917/1723793820587-1e7ce1eb-9f22-4341-889d-b8d97bf7692a.webp)

2、选择设备类型以及电源状态，输入对应的IP地址、用户名以及密码。

![](https://cdn.nlark.com/yuque/0/2024/webp/38747917/1723793820678-1890b436-2f5b-455b-81d9-d4b7bf960500.webp)

3、提示远程主机证书存在问题，点击忽略。

![](https://cdn.nlark.com/yuque/0/2024/webp/38747917/1723793820857-62b8e38d-e8fe-49a5-b435-569d429407fd.webp)

4、为新虚拟机选择主机。

![](https://cdn.nlark.com/yuque/0/2024/webp/38747917/1723793820955-282dc6c1-630f-4efa-bfe7-5f6e4d92dd7a.webp)

5、提示远程主机证书存在问题，点击忽略。

![](https://cdn.nlark.com/yuque/0/2024/webp/38747917/1723793821033-0e12b84b-c875-4895-a247-010789d7cdf8.webp)

6、输入目标VM名称以及目标VM文件夹,点击下一步。

![](https://cdn.nlark.com/yuque/0/2024/webp/38747917/1723793821092-27a0b6b4-3871-49fd-a3b0-1e444813f2a0.webp)

7、选择新虚拟机的位置。

![](https://cdn.nlark.com/yuque/0/2024/webp/38747917/1723793821079-41ee34d5-36c7-4418-8e98-d4cc5bc8b999.webp)

8、点击“Data to copy”后面的“Edit”可以进入编辑。

![](https://cdn.nlark.com/yuque/0/2024/webp/38747917/1723793821278-3cb0934b-29f1-4617-925e-9c0b7419df9b.webp)

9、在此界面可以根据自己需要选择要迁移的磁盘（实验环境我这里就不做更改了），点击“Advanced”切换到高级功能模块。

![](https://cdn.nlark.com/yuque/0/2024/webp/38747917/1723793821423-c293eb6e-a8e6-431b-8b53-5fba1fa008d0.webp)

10、点击“Destination Layout”,在磁盘类型中选择精简置备（Thin），点击下一步。

![](https://cdn.nlark.com/yuque/0/2024/webp/38747917/1723793821449-9be4f921-c26f-454c-9da8-bdcc9bd7a40b.webp)

11、点击“finish”进入P2V任务。

![](https://cdn.nlark.com/yuque/0/2024/webp/38747917/1723793821517-65be4461-7767-4c37-bd5f-859b075fd1c6.webp)

12、可以在控制台看到这个任务，完成后能够查看任务详细信息。

![](https://cdn.nlark.com/yuque/0/2024/webp/38747917/1723793821577-ed10533b-618f-41ac-aa7f-c59f6a7ac3b8.webp)

13、源系统信息。

![](https://cdn.nlark.com/yuque/0/2024/webp/38747917/1723793821652-274ccd18-9db6-406a-98cf-a3fdb1b1f78f.webp)

14、P2V后VM信息，169.254.195.254/16是因为没有设置IP地址的缘故。

![](https://cdn.nlark.com/yuque/0/2024/webp/38747917/1723793822067-92b8e9f7-187a-4eaa-8011-d574b871d447.webp)

# **第6章对 linux系统设备进行P2V操作**** **** **
1、点击“Convert machine”进行PV2操作。

![](https://cdn.nlark.com/yuque/0/2024/webp/38747917/1723793822137-49c41036-e4f8-41be-b132-588d3e3819f0.webp)

2、选择主机类型为“linux”，填写好相应的IP地址、账号以及密码。

![](https://cdn.nlark.com/yuque/0/2024/webp/38747917/1723793822078-e69cc138-0ba3-465e-a398-f253e201c032.webp)

3、点击“yes”。

![](https://cdn.nlark.com/yuque/0/2024/webp/38747917/1723793822147-b12c8514-557f-4cda-86e4-3e892c3d970e.webp)

4、为新虚拟机选择主机，填写好IP地址、用户名以及密码。

![](https://cdn.nlark.com/yuque/0/2024/webp/38747917/1723793822177-2f069d57-b35f-4e7b-bf98-41d1516c20cb.webp)

5、提示远程主机证书存在问题，点击忽略。

![](https://cdn.nlark.com/yuque/0/2024/webp/38747917/1723793822411-153655bc-cd17-4cb6-85a6-5af50a3a02d5.webp)

6、填写目标VM的名称以及目标文件夹。

![](https://cdn.nlark.com/yuque/0/2024/webp/38747917/1723793822539-9cab541a-634e-4bd5-bad6-0b18b51e210e.webp)

7、选择新虚拟机的位置。

![](https://cdn.nlark.com/yuque/0/2024/webp/38747917/1723793822577-9d7b9e06-ca4a-4aff-8f2a-9613a186ebed.webp)

8、点击“Data to copy”后面的“Edit”可以进入编辑。

![](https://cdn.nlark.com/yuque/0/2024/webp/38747917/1723793822557-e88df7d3-36b3-4f26-8408-112875514053.webp)

9、在此界面可以根据自己需要选择要迁移的磁盘（实验环境我这里就不做更改了），点击“Advanced”切换到高级功能模块。

![](https://cdn.nlark.com/yuque/0/2024/webp/38747917/1723793822789-194a602f-8543-4a83-a269-4d51b81ad3f7.webp)

10、点击“Destination Layout”,在磁盘类型中选择精简置备（Thin），点击下一步。

![](https://cdn.nlark.com/yuque/0/2024/webp/38747917/1723793823068-6607f6e7-2d30-4bbc-bacc-4a074f534150.webp)

11、点击“finish”开始PV2任务。

![](https://cdn.nlark.com/yuque/0/2024/webp/38747917/1723793823044-a2beb082-cde8-4905-bf36-6e4952acc495.webp)

12、此时，可以在控制台看到PV2任务。

![](https://cdn.nlark.com/yuque/0/2024/webp/38747917/1723793823104-acd49203-5582-48d9-b021-9a2abbc0c662.webp)

13、点击该任务可以查看详细信息。

![](https://cdn.nlark.com/yuque/0/2024/webp/38747917/1723793823128-2075c729-71aa-4f9b-80ca-8b3e07f82e14.webp)

到此PV2操作就完成成功了。





