# Socket套接字编程

套接字编程（Socket Programming）是计算机网络编程的基础，它允许不同计算机上的应用程序通过网络进行通信。套接字是操作系统提供的一个抽象层，使得开发者能够通过标准化的接口来实现网络通信，而不必关心底层的网络硬件和协议细节。套接字主要分为两种类型：流式套接字（TCP）和数据报套接字（UDP）。下面将详细介绍套接字编程的基本概念和步骤，以最常见的TCP套接字为例进行说明。

## 套接字类型

* TCP（Transmission Control Protocol）套接字：提供面向连接的、可靠的、基于字节流的服务。数据在传输前需要建立连接，且保证数据的顺序和完整性。
* UDP（User Datagram Protocol）套接字：提供无连接的、不可靠的、基于数据报的服务。数据发送不需要建立连接，但不保证数据的顺序和到达。

## 套接字编程步骤（以TCP为例）

### 服务器端

1. 创建套接字：使用socket()函数创建一个套接字，指定为SOCK_STREAM（TCP）类型，以及使用的协议族（通常是AF_INET，代表IPv4）。

2. 绑定端口：通过bind()函数将套接字与一个IP地址和端口号绑定，使该套接字准备接受连接。

3. 监听连接：使用listen()函数让套接字进入监听状态，开始监听客户端的连接请求，参数指定最大等待连接数。

4. 接受连接：调用accept()函数阻塞等待，直到有客户端连接请求到达，然后接受连接并返回一个新的套接字，专门用于与这个客户端通信。

5. 收发数据：通过新创建的套接字使用send()和recv()函数与客户端交换数据。

6. 关闭连接：数据交换完成后，使用close()函数关闭套接字，释放资源。

### 客户端

1. 创建套接字：同样使用socket()函数创建一个TCP类型的套接字。

2. 连接服务器：通过connect()函数尝试与服务器建立连接，需要指定服务器的IP地址和端口号。

3. 收发数据：一旦连接成功，就可以通过这个套接字使用send()和recv()函数与服务器交换数据。

4. 关闭连接：数据交换完毕后，调用close()函数关闭套接字。

### C++示例

* 服务器端代码

```cpp
#include <iostream>
#include <string>
#include <boost/asio.hpp>

using boost::asio::ip::tcp;

int main() {
    try {
        boost::asio::io_service io_service;

        tcp::acceptor acceptor(io_service, tcp::endpoint(tcp::v4(), 12345));

        for (;;) {
            tcp::socket socket(io_service);
            acceptor.accept(socket);

            std::string message = "Hello from server";

            boost::system::error_code ignored_error;
            boost::asio::write(socket, boost::asio::buffer(message), ignored_error);
        }
    } catch (std::exception& e) {
        std::cerr << e.what() << std::endl;
    }

    return 0;
}
```

注意：上述示例使用了Boost.Asio库，因为它提供了高层次的异步I/O功能，简化了网络编程。如果你想使用标准库进行网络编程，可以参考下面的标准库示例。

* 客户端代码

```cpp
#include <iostream>
#include <boost/array.hpp>
#include <boost/asio.hpp>

using boost::asio::ip::tcp;

int main() {
    try {
        boost::asio::io_service io_service;

        tcp::resolver resolver(io_service);
        tcp::resolver::query query("localhost", "12345");
        tcp::resolver::iterator endpoint_iterator = resolver.resolve(query);

        tcp::socket socket(io_service);
        boost::asio::connect(socket, endpoint_iterator);

        boost::array<char, 128> buf;
        boost::system::error_code error;

        size_t len = socket.read_some(boost::asio::buffer(buf), error);

        if (!error) {
            std::cout.write(buf.data(), len);
        } else {
            std::cerr << "Error: " << error.message() << std::endl;
        }
    } catch (std::exception& e) {
        std::cerr << e.what() << std::endl;
    }

    return 0;
}
```

这两个示例展示了如何使用Boost.Asio库在C++中实现一个简单的TCP服务器和客户端。服务器监听12345端口，接收连接并发送一条消息给客户端；客户端则连接到服务器并打印接收到的消息。如果希望使用C++标准库（STL）进行网络编程，可以探索使用`std::chrono`、`std::thread`配合`<sys/socket.h>`等系统调用，但其编程复杂度相对较高，且不直接提供异步I/O支持。
