# 项目简介
基于 Linux 操作系统构建服务器集群时，一个共享的文件系统是常见的基础实施。共享存储设备是昂贵的且容易成为瓶颈。如果每台服务器都共享出一定存储空间形成共享存储池，再用这个存储池构建一个共享的文件系统，即可降低成本，也可得到更好的可靠性。本项目的目的就是构建这样一个共享文件系统。
# 技术架构
SpringBoot + SpringCloudAlibaba+Redis + MySQL+RocketMQ
# 快速开始
本项目分为client和dataserver两大模块
dataserver：接收来自client的上传和下载请求，完成文件块的储存
client：负责文件的分块与组装，并向datserver发出上传和下载请求
