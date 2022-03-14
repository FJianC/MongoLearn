# Mongo C 和 C++ 驱动的编译

[mongo-c-driver 官方教程](http://mongoc.org/libmongoc/current/installing.html)  
[mongo-cxx-driver 官方教程](http://mongocxx.org/mongocxx-v3/installation/)

## 下载 mongo-c-driver 和 mongo-cxx-driver 的源码

**两种方式**  
1. git 拉取
`git clone https://github.com/mongodb/mongo-c-driver.git`
`git clone https://github.com/mongodb/mongo-cxx-driver.git`
2. 指定版本
`https://github.com/mongodb/mongo-c-driver/releases`
`https://github.com/mongodb/mongo-cxx-driver/releases`

## 编译依赖

安装 cmake openssl  
`sudo apt install cmake libssl-dev libsas2-dev`

## 编译 mongo-c-driver

``` shell
cd mongo-c-driver
mkdir build
cd build
cmake -DENABLE_AUTOMATIC_INIT_AND_CLEANUP=OFF ..
cmake --build .
sudo cmake --build . --target install
```

安装后会生成 libmongoc 和 libbson 库  
`-DCMAKE_BUILD_TYPE=Release` 指定编译版本  
`-DCMAKE_INSTALL_RPATH=/usr/local` 指定安装路径，默认路径 /usr/local  
`-DENABLE_TESTS=OFF` 是否编译测试样例  
其他 cmake 选项列表可执行 `cmake -L ..` 查看  

## 编译 mongo-cxx-driver

mongo-cxx-driver 编译依赖 libmongoc 库  
需要先编译 mongo-c-driver 或者直接安装 `sudo apt install libmongoc-1.0.0`

``` shell
cd mongo-cxx-driver
mkdir build
cd build
cmake .. -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/usr/local
sudo cmake --build . --target EP_mnmlstc_core
cmake --build .
sudo cmake --build . --target install
```

安装后会生成 libmongocxx 和 libbsoncxx 库  
`-DCMAKE_BUILD_TYPE=Release` 指定编译版本  
`-DCMAKE_INSTALL_RPATH=/usr/local` 指定安装路径，默认路径 /usr/local  
`-DENABLE_TESTS=OFF` 是否编译测试样例  
其他 cmake 选项列表可执行 `cmake -L ..` 查看  

# Mongo C++ 驱动代码测试

[mongo-c-driver API文档](http://mongoc.org/libmongoc/current/api.html)  
[mongo-cxx-driver API文档](http://mongocxx.org/api/current/)

``` cpp
#include <iostream>

#include <bsoncxx/json.hpp>
#include <mongocxx/client.hpp>
#include <mongocxx/instance.hpp>

using bsoncxx::builder::basic::kvp;
using bsoncxx::builder::basic::make_document;

int main(int argc, char* argv[]) {
    mongocxx::instance instance{}; // This should be done only once.
    mongocxx::client client{mongocxx::uri{mongocxx::uri::k_default_uri}};

    auto test = client["test"];
    auto result = test.run_command(make_document(kvp("ping", 1)));
    std::cout << bsoncxx::to_json(result) << std::endl;

    return 0;
}
```

**编译**
``` shell
# 不使用 pkg-config 和 CMake 编译
c++ --std=c++11 test.cpp -o test -I/usr/local/include/mongocxx/v_noabi -I/usr/local/include/bsoncxx/v_noabi -L/usr/local/lib -lmongocxx -lbsoncxx
# 借助 pkg-config 编译
c++ --std=c++11 test.cpp -o test $(pkg-config --cflags --libs libmongocxx)
# 借助 CMake 进行编译
# 需要写 CMakeFile 详情看官方文档
```

可能会遇到的问题：`libxxx: cannot open shared object file: No such file or directory`  
例如：找不到 libmongocxx.so._noabi  
1. 执行 `ldd libmongocxx.so._noabi`，会提示 libmongocxx.so._noabi not find
2. 查找 libmongocxx.so._noabi 所在的路径：  
`sudo find / -name libmongocxx.so._noabi`
3. 找到之后查看文件信息：  
`ls -l /usr/local/lib/libmongocxx.so._noabi`  
发现是链接文件，指向了真实文件
4. 删了链接文件重新进行链接：  
`sudo ln -s /usr/local/lib/libmongocxx.so.3.4.0 /usr/local/lib/libmongocxx.so._noabi`
5. 再重新加载库：  
`sudo ldconfig`
6. 检查是否成功：  
`ldd libmongocxx.so._noabi`
