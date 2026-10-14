# RDMA Hello World

基于 C 和 libibverbs 的 RDMA 通信学习实验，包含 Send/Receive、RDMA Read、RDMA Write，以及双缓冲区 scatter/gather 发送实验。当前默认分支为 `client`。

本仓库适合用于理解 RDMA 资源管理和通信流程，当前版本仍包含实验代码；尚未提供可复现的性能测试结果。

## 通信流程

1. 创建设备上下文、Protection Domain、Memory Region、Completion Queue 和 RC Queue Pair。
2. 通过 TCP 交换 LID、QPN 和 PSN，将 QP 从 INIT 转换为 RTR、RTS。
3. 客户端发送消息，并交换远端内存地址与 rkey。
4. 服务端发起 RDMA Read；客户端随后发起 RDMA Write，并通过 Send 通知服务端。
5. 轮询 CQ，检查 Work Completion 状态。

## 代码结构

| 文件 | 用途 |
| --- | --- |
| `client.c` | 客户端消息发送、内存信息交换及 RDMA Write |
| `server.c` | 服务端消息接收及 RDMA Read |
| `setup_ib.c` / `setup_ib.h` | TCP 握手、设备与资源初始化、QP 状态转换及清理 |
| `helper.c` / `helper.h` | Work Request 提交及 CQ 轮询 |
| `config.h` | TCP 端口、IB 端口、MTU、队列和缓冲区配置 |
| `Makefile` | 编译客户端和服务端 |

## 环境要求

- Linux、C 编译器、Make，以及提供 `infiniband/verbs.h` 和 `libibverbs` 的开发环境。
- 两端可用的 RDMA 设备、驱动与网络配置。
- TCP 握手端口默认 `8080`，IB 端口默认 `1`，MTU 配置为 `4096`，缓冲区大小为 `1024` 字节。

当前地址路径使用 LID，且 `is_global = 0`；直接运行以匹配的 InfiniBand 环境为前提。RoCE/GID 路径需要额外适配和验证。

## 编译与启动

```sh
git clone --branch client https://github.com/B-ay/rdma-hello-world.git
cd rdma-hello-world
make
```

先在服务端启动：

```sh
./server
```

再在客户端启动：

```sh
./client <服务端主机名或IP>
```

上述步骤来自当前 Makefile 和程序入口。本次文档整理没有在 RDMA 硬件上验证运行结果。

## 当前限制与后续工作

- 当前 `post_send1` 将两个 SGE 的长度固定为 3 和 4 字节，不能将首次发送描述为完整 Hello World 消息验证。
- 服务端使用 `printf(ib_res.buf)`，后续应改为带长度边界的安全输出，并检查消息终止条件。
- 初始化失败后程序仍可能继续执行；部分请求提交函数未传播错误，需要补全失败退出与错误处理。
- TCP 握手尚需处理短读、短写、断连及结构体序列化兼容性。
- 第二个 Memory Region 和缓冲区尚需补全清理。
- 后续补充数据正确性检查、运行环境记录、延迟与吞吐测试，再据此报告性能。

## 来源与贡献说明

现有源文件保留了 `Author: Arie` 的署名，仓库包含 MIT 许可证。个人改动范围需要结合提交记录及作者说明确认；引用或展示时请保留现有署名和许可说明。