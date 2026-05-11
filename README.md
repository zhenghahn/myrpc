# MyRpc

一个基于 C++ 实现的高性能 RPC（Remote Procedure Call）框架，支持服务注册与发现、负载均衡和高可用特性。

## 功能特性

- ✅ **高性能通信**：基于 Muduo 网络库实现 Reactor 模式
- ✅ **服务注册与发现**：集成 Zookeeper 作为注册中心
- ✅ **高效序列化**：使用 Protobuf 实现数据序列化/反序列化
- ✅ **自定义协议**：设计自定义协议解决 TCP 粘包/拆包问题
- ✅ **负载均衡**：支持多种负载均衡策略
- ✅ **高可用**：Watcher 机制动态感知服务状态

## 技术栈

- **语言**：C++ 11
- **网络库**：Muduo
- **序列化**：Google Protobuf
- **服务注册**：Apache Zookeeper
- **构建工具**：CMake

## 项目结构

```
MyRpc/
├── src/                    # 核心源代码
│   ├── Krpcchannel.cc      # 客户端通信通道
│   ├── Krpcprovider.cc     # 服务端提供者
│   ├── Krpccontroller.cc   # RPC 控制器
│   ├── Krpcapplication.cc  # 应用入口
│   ├── Krpcconfig.cc       # 配置管理
│   ├── zookeeperutil.cc    # Zookeeper 工具类
│   └── include/            # 头文件目录
├── example/                # 示例代码
│   ├── caller/             # 客户端示例
│   └── callee/             # 服务端示例
├── build/                  # 构建输出目录
├── bin/                    # 可执行文件输出
└── CMakeLists.txt          # CMake 配置文件
```

## 快速开始

### 环境要求

- Linux 操作系统
- CMake 3.0+
- Protobuf 3.0+
- Muduo 库
- Zookeeper 3.4+

### 编译安装

```bash
# 克隆项目
git clone https://github.com/yourname/MyRpc.git
cd MyRpc

# 创建构建目录
mkdir build && cd build

# 生成构建文件
cmake ..

# 编译
make -j4

# 安装（可选）
make install
```

### 运行示例

#### 1. 启动 Zookeeper

```bash
zkServer.sh start
```

#### 2. 启动服务端

```bash
cd bin
./provider
```

#### 3. 启动客户端

```bash
cd bin
./client
```

## 使用示例

### 定义服务接口

```protobuf
// user.proto
syntax = "proto3";

package Kuser;

message LoginRequest {
    string name = 1;
    string pwd = 2;
}

message LoginResponse {
    bool success = 1;
    string token = 2;
}

service UserServiceRpc {
    rpc Login(LoginRequest) returns (LoginResponse);
}
```

### 服务端实现

```cpp
class UserService : public Kuser::UserServiceRpc {
public:
    void Login(google::protobuf::RpcController* controller,
               const Kuser::LoginRequest* request,
               Kuser::LoginResponse* response,
               google::protobuf::Closure* done) {
        // 业务逻辑实现
        std::string name = request->name();
        std::string pwd = request->pwd();
        
        bool success = (name == "admin" && pwd == "123456");
        response->set_success(success);
        
        done->Run();
    }
};

int main() {
    KrpcApplication::Init(argc, argv);
    KrpcProvider provider;
    provider.NotifyService(new UserService());
    provider.Run();
    return 0;
}
```

### 客户端调用

```cpp
int main() {
    KrpcApplication::Init(argc, argv);
    
    Kuser::UserServiceRpc_Stub stub(new KrpcChannel());
    
    Kuser::LoginRequest request;
    request.set_name("admin");
    request.set_pwd("123456");
    
    Kuser::LoginResponse response;
    KrpcController controller;
    
    stub.Login(&controller, &request, &response, nullptr);
    
    if (controller.Failed()) {
        std::cout << controller.ErrorText() << std::endl;
    } else {
        std::cout << "Login success: " << response.success() << std::endl;
    }
    
    return 0;
}
```

## 技术架构

### 架构图

```
┌─────────────────────────────────────────────────────────────────────┐
│                        客户端 (Client)                              │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────────────┐   │
│  │  Service Stub│───►│ KrpcChannel  │───►│      Socket         │   │
│  │  (代理对象)   │    │  (序列化)    │    │   (网络传输)        │   │
│  └──────────────┘    └──────────────┘    └──────────┬───────────┘   │
│                                                      │              │
└───────────────────────────────────────────────────────┼──────────────┘
                                                        │
                    ┌───────────────────────────────────┼───────────────────┐
                    │                                 ▼                   │
                    │  ┌──────────────────────────────────────────────┐   │
                    │  │            Zookeeper (服务注册中心)          │   │
                    │  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  │   │
                    │  │  │Service A │  │Service B │  │Service C │  │   │
                    │  │  └────┬─────┘  └────┬─────┘  └────┬─────┘  │   │
                    │  │       │             │             │       │   │
                    │  └───────┼─────────────┼─────────────┼───────┘   │
                    │         │             │             │           │
                    │         ▼             ▼             ▼           │
                    │  ┌──────────┐  ┌──────────┐  ┌──────────┐     │   │
                    │  │ Provider1│  │ Provider2│  │ Provider3│     │   │
                    │  │ (Server) │  │ (Server) │  │ (Server) │     │   │
                    │  └──────────┘  └──────────┘  └──────────┘     │   │
                    │                                                │   │
                    └───────────────────────┬────────────────────────┘   │
                                            │                          │
                                            ▼                          │
              ┌─────────────────────────────────────────────────────────┐ │
              │                    服务端 (Server)                      │ │
              │  ┌──────────────┐    ┌──────────────┐    ┌───────────┐ │ │
              │  │ Socket       │───►│ KrpcProvider │───►│Service Imp│ │ │
              │  │ (网络接收)   │    │ (反序列化)   │    │(业务逻辑) │ │ │
              │  └──────────────┘    └──────────────┘    └───────────┘ │ │
              └─────────────────────────────────────────────────────────┘ │
                    └───────────────────────────────────────────────────┘
```

### 核心组件

| 组件 | 职责 |
|------|------|
| **KrpcChannel** | 客户端通信通道，负责序列化和网络传输 |
| **KrpcProvider** | 服务端提供者，负责服务注册和请求处理 |
| **KrpcController** | RPC 控制器，管理调用状态和错误信息 |
| **ZkClient** | Zookeeper 客户端，实现服务注册与发现 |
| **KrpcApplication** | 应用入口，初始化配置和环境 |

### 协议格式

```
请求协议: [4B Total Len] + [4B Header Len] + [Header] + [Args]
响应协议: [4B Total Len] + [Response Data]
```

## 许可证

MIT License

## 贡献

欢迎提交 Issue 和 Pull Request！

## 联系方式

如有问题或建议，请通过以下方式联系：
- 邮箱：your@email.com
- GitHub：https://github.com/yourname/MyRpc