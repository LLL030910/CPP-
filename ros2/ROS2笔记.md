`ros2 run 功能包 可执行文件`

运行节点的原理：寻找 `AMENT_PREFIX_PATH` 环境变量下的路径
默认 `AMENT_PREFIX_PATH=/opt/ros/humble`

### 2.3使用功能包组织C++节点

#### 2.3.1 创建包

`$ ros2 pkg create demo_cpp_pkg --build-type ament_cmake --license Apache-2.0`

<img src="C:/Users/13489/AppData/Roaming/Typora/typora-user-images/image-20260929215741872.png" alt="image-20260929215741872" style="zoom:50%;" />

#### 2.3.2 编写代码

在 `demo_cpp_pkg/src/cpp_node.cpp` 中编写代码

```cpp
#include "rclcpp/rclcpp.hpp"

int main(int argc, char ** argv) {
    rclcpp::init(argc, argv);
    auto node = std::make_shared<rclcpp::Node>("cpp_node");
    RCLCPP_INFO(node->get_logger(), "hello, C++ node!");
    rclcpp::spin(node);
    rclcpp::shutdown();
    return 0;
}
```

#### 2.3.3 节点注册/CMakeLists.txt

在 `CMakeLists.txt` 中添加：

```cmake
find_package(rclcpp REQUIRED)
add_executable(cpp_node src/cpp_node.cpp) # (可执行文件名 源文件路径)

# target_include_directories(cpp_node PUBLIC ${rclcpp_INCLUDE_DIRS})
# target_link_libraries(cpp_node ${rclcpp_LIBRARIES})
ament_target_dependencies(cpp_node rclcpp) # 这句可以替代上面两行

# 将可执行文件 cpp_node 拷贝到 instal/lib目录下
install(TARGETS cpp_node
  DESTINATION lib/${PROJECT_NAME}
)
```

#### 2.3.4 声明

在 `package.xml` 中添加：`<depend>rclcpp</depend>`

#### 2.3.5 构建

在项目目录下构建：

`ros2@ros2-virtual-machine:~/chapt2$ colcon build`

#### 2.3.6 运行

先更改环境变量 `AMENT_PREFIX_PATH`

```shell
ros2@ros2-virtual-machine:~/chapt2$ source install/setup.sh
ros2@ros2-virtual-machine:~/chapt2$ printenv | grep AMENT
AMENT_PREFIX_PATH=/home/ros2/chapt2/install/demo_python_pkg:/home/ros2/chapt2/install/demo_cpp_pkg:/opt/ros/humble
```

运行：

`ros2 run demo_cpp_pkg cpp_node`

### 2.4 多功能包最佳实践

`colcon build` 构建工作区下的所有pkg，命令在哪个目录下执行生成的 `build` `install` `log` 就在哪个目录。

`colcon build --packages-select [PKG_NAME ...]` 构建指定的功能包

#### 2.4.1 指定构建顺序

如果想要先构建 `cpp`功能包，再构建 `python` 功能包，则需要在python功能包下的 `package.xml` 增加依赖

```cmake
<depend>demo_cpp_pkg</depend>
```

### 2.5 ROS2 基础之编程

#### 2.5.1 运行Python节点

**1.注册**

在`setup.py` 文件中进行注册

```cmake
entry_points={
        'console_scripts': [
            'python_node = demo_python_pkg.python_node:main',
            'person_node = demo_python_pkg.person_node:main', # 增加节点
        ],
    },
```

**2.构建** 功能包 `colcon build`， 构建完后 `install` 目录下会出现可执行文件，但还需更改环境变量才能运行。

<img src="C:/Users/13489/AppData/Roaming/Typora/typora-user-images/image-20261003120730792.png" alt="image-20261003120730792" style="zoom:50%;" />

**3.更改环境变量**

```shell
ros2@ros2-virtual-machine:~/chapt2/chapt2_ws$ source install/setup.bash
```

#### 2.5.1 编写C++节点

1. 包含 `rclcpp` 头文件

	`vscode`编辑器找不到头文件则需要添加 `include`路径: `/opt/ros/humble/include/**`

2. 编写代码

3. CMakeLists.txt

	```cmake
	add_executable(person_node src/person_node.cpp) # 单个文件想要单独运行添加可执行文件
	ament_target_dependencies(person_node rclcpp)   # 找（链接）头文件和库文件
	install(TARGETS cpp_node person_node            # 将可执行文件拷贝到 instal/lib目录下 
	  DESTINATION lib/${PROJECT_NAME}
	)                                           
	```

4. 更改环境变量

```shell
ros2@ros2-virtual-machine:~/chapt2/chapt2_ws$ printenv | grep AMENT
AMENT_PREFIX_PATH=/opt/ros/humble
ros2@ros2-virtual-machine:~/chapt2/chapt2_ws$ source install/setup.bash
ros2@ros2-virtual-machine:~/chapt2/chapt2_ws$ printenv | grep AMENT
AMENT_PREFIX_PATH=/home/ros2/chapt2/chapt2_ws/install/demo_python_pkg:/home/ros2/chapt2/chapt2_ws/install/demo_cpp_pkg:/opt/ros/humble
```

​	5. 运行节点 `ros2 run demo_cpp_pkg person_node`

#### 2.5.2 C++新特性

##### `shared` 共享指针

```cpp
#include <iostream>
#include <memory>

int main() {
    auto p1 = std::make_shared<std::string>("This is a str.");
    std::cout << "p1 use count: " << p1.use_count() << " p1 point address: " << p1.get() << std::endl; // 1
    
    auto p2 = p1;
    std::cout << "p1 use count: " << p1.use_count() << " p1 point address: " << p1.get() << std::endl; // 2
    std::cout << "p2 use count: " << p2.use_count() << " p2 point address: " << p2.get() << " *p2: " << *p2 << std::endl; // 2

    p1.reset();  // release p1's pointer
    std::cout << "p1 use count: " << p1.use_count() << " p1 point address: " << p1.get() << std::endl; // 0
    std::cout << "p2 use count: " << p2.use_count() << " p2 point address: " << p2.get() << " *p2: " << *p2 << std::endl; // 1
}
```

```shell
p1 use count: 1 p1 point address: 0x5c7fb2befec0
p1 use count: 2 p1 point address: 0x5c7fb2befec0
p2 use count: 2 p2 point address: 0x5c7fb2befec0 *p2: This is a str.
p1 use count: 0 p1 point address: 0
p2 use count: 1 p2 point address: 0x5c7fb2befec0 *p2: This is a str.
```

##### Lambda表达式

\[捕获列表\]\(参数列表\)->返回类型 {函数体};

##### functional 函数包装器

```cpp
#include <iostream>
#include <functional>

void save_with_free_func(const std::string & file_name) {
    std::cout << "free function: " << file_name << std::endl;
}

class SaveFile {
public:
    SaveFile() = default;
    ~SaveFile() = default;
    void save_with_member_func(const std::string & file_name) {
        std::cout << "member function: " << file_name << std::endl;
    }
};

int main() {
    SaveFile save_file;
    auto save_with_lambda_func = [](const std::string & file_name) -> void {
        std::cout << "lambda function: " << file_name << std::endl;
    };

    // save_with_free_func("file.txt");
    // save_file.save_with_member_func("file.txt");
    // save_with_lambda_func("file.txt");

    std::function<void(const std::string&)> save1 = save_with_free_func;
    std::function<void(const std::string&)> save2 = save_with_lambda_func;
    std::function<void(const std::string&)> save3 = std::bind(&SaveFile::save_with_member_func, &save_file, std::placeholders::_1);

    save1("file.txt");
    save2("file.txt");
    save3("file.txt");
}
```

#### 2.5.3 多线程与回调函数

现在任务为下载3个页面的小说，然后统计小说的字数。可以使用多线程+回调函数实现：创建3个线程，然后下载完后调用对应的统计字数函数。

##### 2.5.3.1 Python版本

```python
import threading
import requests

class DownLoader:
    def download(self, url, callback_word_count):
        print(f"thread ID: {threading.get_ident()} start downlowd url: {url}")
        response = requests.get(url)
        response.encoding = 'utf-8'
        callback_word_count(url, response.text)

    # 创建线程，target为真正执行的函数
    def start_download(self, url, callback_word_count):
        thread = threading.Thread(target=self.download, args=(url, callback_word_count))
        thread.start()
        
def word_count(url, result):
    print(f'{url}, {len(result)} char, context: {result[:5]}')

def main():
    downloader = DownLoader()
    downloader.start_download("http://127.0.0.1:8000/novel1.txt", word_count)
    downloader.start_download("http://127.0.0.1:8000/novel2.txt", word_count)
    downloader.start_download("http://127.0.0.1:8000/novel3.txt", word_count)
```

使用 `python3 -m http.server` 启动当前目录服务器

##### 2.5.3.2 C++版本

需要使用到 `httplib` 库，函数包装器，回调函数

需要在 `CMakeLists.txt` 中包含 `cpp-httplib` : `include_directories(include)`

```c++
#include <iostream>
#include <chrono> // the head file of time
#include <thread>
#include <functional>
#include "cpp-httplib/httplib.h"

class DownLoader
{
public:
    // 参数回调函数使用函数包装器
    void downLoad(const std::string &host, const std::string &path,
                  std::function<void(const std::string &, const std::string &)> callback)
    {
        std::cout << "thred ID: " << std::this_thread::get_id() << std::endl;
        httplib::Client client(host);
        auto response = client.Get(path);
        if (response && response->status == 200)
        {
            callback(path, response->body);
        }
    }
	// 创建线程，目标函数为downLoad
    void start_down(const std::string &host, const std::string &path,
                    std::function<void(const std::string &, const std::string &)> callback)
    {
        auto download_func = std::bind(&DownLoader::downLoad, this, std::placeholders::_1, std::placeholders::_2, std::placeholders::_3);
        // arg1:目标函数, 后续为参数
        std::thread thread(download_func, host, path, callback);
        // std::thread thread(downLoad, host, path, callback);  
        thread.detach();
    }
};

int main()
{
    DownLoader d = DownLoader();
    auto word_count = [](const std::string & url, const std::string& result) -> void 
    {
        std::cout << url << " download finish : " << result << " length: " << result.length() << std::endl;
    };

    d.start_down("http://127.0.0.1:8000", "/novel1.txt", word_count);
    d.start_down("http://127.0.0.1:8000", "/novel2.txt", word_count);
    d.start_down("http://127.0.0.1:8000", "/novel3.txt", word_count);

    std::this_thread::sleep_for(std::chrono::milliseconds(1000 * 10));
}
```

## 3. 话题

### 3.1 话题通信

#### 1. `ros2 topic list`

```shell
ros2 topic list              # 只列话题名
ros2 topic list -t           # 同时显示消息类型
ros2 topic list -c           # 统计话题数量
```

#### 2. `ros2 topic info`

查看话题详细信息（查看类型、发布者数量、订阅者数量）
`ros2 topic info /turtle1/cmd_vel`

#### 3.查看消息类型结构

```shell
ros2 interface show turtlesim/msg/Pose          # 查看类型结构
ros2@ros2-virtual-machine:~$ ros2 interface show geometry_msgs/msg/Twist
# This expresses velocity in free space broken into its linear and angular parts.

Vector3  linear
	float64 x
	float64 y
	float64 z
Vector3  angular
	float64 x
	float64 y
	float64 z
```

#### 4. 打印话题数据

1. **订阅：** ROS 2 创建了一个临时的“订阅者”节点，去订阅 `turtlesim/msg/Pose` 这个话题。
2. **接收：** 只要你的发布者节点（`/turtle1/pose`）往这个话题里发了一条消息，这个临时节点就会立刻收到。
3. **打印：** 收到后，它把消息里的数据结构（比如你自定义的 ``/turtle1/poses` 里的字段内容），以 YAML 格式（一种易读的文本格式）实时打印在屏幕上。

```shell
ros2 topic echo /turtle1/pose              # 持续打印
ros2 topic echo /turtle1/pose --once       # 只打印一条
    x: 5.544444561004639
    y: 5.544444561004639
    theta: 0.0                # 角度（弧度制）
    linear_velocity: 0.0      # 线速度
    angular_velocity: 0.0     # 角速度（逆时针为+）
```

查看节点列表: `ros2 node list`

查看节点信息: `ros2 node info /turtlesim`

```shell
ros2@ros2-virtual-machine:~$ ros2 node info /turtlesim
/turtlesim
  Subscribers:
    /parameter_events: rcl_interfaces/msg/ParameterEvent
    /turtle1/cmd_vel: geometry_msgs/msg/Twist       # 话题名字：话题类型（消息接口）
  Publishers:
    /parameter_events: rcl_interfaces/msg/ParameterEvent
    /rosout: rcl_interfaces/msg/Log
    /turtle1/color_sensor: turtlesim/msg/Color
    /turtle1/pose: turtlesim/msg/Pose
```

对话题发布消息 `ros2 topic pub 话题名字 消息接口 `

```shell
ros2@ros2-virtual-machine:~$ ros2 topic pub /turtle1/cmd_vel geometry_msgs/msg/Twist 
"{linear: {x: 0.5, y: 0.0}, angular: {z: 0.0}}"
publisher: beginning loop
publishing #1: geometry_msgs.msg.Twist(linear=geometry_msgs.msg.Vector3(x=0.5, y=0.0, z=0.0), angular=geometry_msgs.msg.Vector3(x=0.0, y=0.0, z=0.0))

publishing #2: geometry_msgs.msg.Twist(linear=geometry_msgs.msg.Vector3(x=0.5, y=0.0, z=0.0), angular=geometry_msgs.msg.Vector3(x=0.0, y=0.0, z=0.0))
```

### 3.2 Python话题订阅与发布

#### 3.2.1 通过话题发布小说

场景：想要通过话题发布小说，且每5s发布小说的一行

<img src="C:/Users/13489/AppData/Roaming/Typora/typora-user-images/image-20261007173116826.png" alt="image-20261007173116826" style="zoom:67%;" />

创建发布者: `create_publisher(msg_type: 消息类型, topic: 话题名字,)` 
创建定时器: `create_timer(timer_period_sec: 秒数, callback: 回调函数)` 
发布消息: `novel_publisher_.publish(msg)` 

```python
import rclpy
from rclpy.node import Node
import requests
from example_interfaces.msg import String
from queue import Queue

class NovelPubNode(Node):
    def __init__(self, node_name):
        super().__init__(node_name=node_name)
        self.get_logger().info(f"{node_name} node is starting!")
        self.novel_queue = Queue()
        self.novel_publisher_ = self.create_publisher(String, "novel_publisher", 10)
        self.create_timer(5, self.timer_callback)
    
    def timer_callback(self):
        if self.novel_queue.qsize() > 0:
			# 包装消息
            line = self.novel_queue.get()
            msg = String()
            msg.data = line
            self.novel_publisher_.publish(msg)
            self.get_logger().info(f"publish: {msg}")

    def download(self, url):
        response = requests.get(url)
        response.encoding = 'utf-8'
        text = response.text
        self.get_logger().info(f"download {url}, length: {len(text)}")
        for line in text.splitlines():
            self.novel_queue.put(line)
```

运行节点之后查看话题列表：

```shell
ros2@ros2-virtual-machine:~/chapt3/topic_ws$ ros2 topic list
/novel_publisher
/parameter_events
/rosout
```

查看话题输出的消息：

```shell
ros2@ros2-virtual-machine:~/chapt3/topic_ws$ ros2 topic echo /novel_publisher 
data: chapter3 hello C++ ROS2
---
data: chapter1 hello python
---
data: chapter2 hello java
---
```

查看话题输出频率：

```shell
ros2@ros2-virtual-machine:~/chapt3/topic_ws$ ros2 topic hz /novel_publisher
average rate: 0.200
        min: 4.999s max: 5.000s std dev: 0.00060s window: 2
average rate: 0.200
        min: 4.999s max: 5.001s std dev: 0.00090s window: 3
```

#### 3.2.2 订阅小说并合成语音

<img src="C:/Users/13489/AppData/Roaming/Typora/typora-user-images/image-20261008143219914.png" alt="image-20261008143219914" style="zoom:33%;" />

下载相关库: `pip3 install espeakng` `pip3 install espeak-ng`

创建订阅者: `create_subscription(msg_type:消息类型, topic:话题名字(需要和发布的名字一模一样), 回调函数, Qos) `
创建专门的线程来进行朗读: `threading.Thread(target=回调函数)`

```python
import espeakng
import rclpy
from rclpy.node import Node
from example_interfaces.msg import String
from queue import Queue
import threading
import time

class NovelSubNode(Node):
    def __init__(self, node_name):
        super().__init__(node_name=node_name)
        self.get_logger().info(f"{node_name} node is starting!")
        self.novel_queue_ = Queue()
        self.novel_suscriber_ = self.create_subscription(String, "novel", self.novel_callback, 10)
        self.speak_thread_ = threading.Thread(target=self.speaker)
        self.speak_thread_.start()
    
    def novel_callback(self, msg):              # msg和发布话题时的msg一致
        self.novel_queue_.put(msg.data)
    
    def speaker(self):
        speaker = espeakng.Speaker()
        while rclpy.ok():
            if self.novel_queue_.qsize() > 0:
                text = self.novel_queue_.get()
                self.get_logger().info(f"speak: {text}")
                speaker.say(text)
                speaker.wait()
            else:
                time.sleep(1)         # 如果当前队列为空，休眠当前线程

def main():
    rclpy.init()
    node = NovelSubNode("novel_sub")
    rclpy.spin(node)
    rclpy.shutdown()
```

### 3.3 C++话题订阅与发布

#### 3.3.1 发布速度控制小海龟画圆

> 需求：控制海龟模拟器中的小海龟转指定半径的圆
>
> 核心问题：
> 1.小海龟凭什么听我的？话题
> 2.前进后退通过单独设置x正负，画圆？通过设置线速度/角速度=半径
> 3.发一下就停了，如何循环发？定时器

创建功能包 `demo_cpp_topic` : `ros2 pkg create demo_cpp_topic --build-type ament_cmake --dependencies rclcpp geometry_msgs turtlesim --license Apache-2.0`
以及节点 `turtle_circle.cpp` 

##### 1. 查看海龟速度指令话题

```shell
ros2@ros2-virtual-machine:~$ ros2 topic info /turtle1/cmd_vel
Type: geometry_msgs/msg/Twist
Publisher count: 0
Subscription count: 1
# 类型：geometry_msgs/msg/Twist
# 作用：控制海龟运动的速度指令话题。发布线速度和角速度即可让海龟移动。
```

##### 2. 编写代码
创建发布者: `create_publisher<geometry_msgs::msg::Twist: 消息类型>("/turtle1/cmd_vel": 话题名字, 10: Qos);`
创建定时器: `create_wall_timer(1000ms, std::bind(&TurtleCircle::time_callback, this): 回调函数);`

```c++
#include "rclcpp/rclcpp.hpp"
#include "geometry_msgs/msg/twist.hpp"
#include <chrono>

using namespace std::chrono_literals;  // 可以在创建定时器时直接传入1000ms

class TurtleCircle : public rclcpp::Node {
public:
    TurtleCircle(const std::string& node_name) : Node(node_name) {
        publisher_ = this->create_publisher<geometry_msgs::msg::Twist>("/turtle1/cmd_vel", 10);
        timer_ = this->create_wall_timer(1000ms, 	std::bind(&TurtleCircle::time_callback, this));
    }
    void time_callback() {
        auto msg = geometry_msgs::msg::Twist();   // 包装消息
        msg.linear.x = 1.0;                       // 线速度x轴方向
        msg.angular.z = 0.5;                      // 角速度z轴（逆时针为+）
        publisher_->publish(msg);                 // 发布消息
    }
private:
    // 声明发布者和定时器
    rclcpp::Publisher<geometry_msgs::msg::Twist>::SharedPtr publisher_;
    rclcpp::TimerBase::SharedPtr timer_;
};

int main(int argc, char ** argv) {
    rclcpp::init(argc, argv);
    auto node = std::make_shared<TurtleCircle>("turtle_circle");
    rclcpp::spin(node);
    rclcpp::shutdown();
    return 0;
}
```

##### 3. CMakeLists.txt

```cmake
add_executable(turtle_circle src/turtle_circle.cpp)   # 增加可执行文件
# 为可执行文件 turtle_circle 添加依赖包，让编译时能找到头文件、链接时能找到库。
ament_target_dependencies(turtle_circle rclcpp geometry_msgs)
# 把编译好的可执行文件 turtle_circle 安装到指定目录
install(TARGETS turtle_circle
  DESTINATION lib/${PROJECT_NAME}
)
```

#### 3.3.2 订阅Pose实现闭环控制

> 需求：告诉小海龟到指定位置，自己滚过去
>
> 核心问题：
> 1.小海龟凭什么听我的？发布话题
> 2.小海龟现在在哪里？订阅 `/turtle1/pose` 话题
> 3.怎么根据当前位置和目标位置计算角速度和线速度？两点之间距离->线速度，当前朝向和目标朝向差->角速度。
>
> x轴为小海龟的朝向（控制前后移动）

查看消息类型 `turtlesim/msg/Pose` 的详细信息

```shell
ros2@ros2-virtual-machine:~$ ros2 interface show turtlesim/msg/Pose
float32 x
float32 y
float32 theta

float32 linear_velocity
float32 angular_velocity
```

获取小海龟当前位置: `create_subscription<turtlesim::msg::Pose: 消息类型>("/turtle1/pose": 话题名字, 10: Qos, std::bind(&TurtleControl::on_pose_received, this, std::placeholders::_1): 回调函数);`

```cpp
#include "rclcpp/rclcpp.hpp"
#include "geometry_msgs/msg/twist.hpp"
#include "turtlesim/msg/pose.hpp"

class TurtleControl : public rclcpp::Node {
public:
    TurtleControl(const std::string& node_name) : Node(node_name) {
        publisher_ = this->create_publisher<geometry_msgs::msg::Twist>("/turtle1/cmd_vel", 10);
        subscriber_ = this->create_subscription<turtlesim::msg::Pose>("/turtle1/pose", 10, std::bind(&TurtleControl::on_pose_received, this, std::placeholders::_1));
    }
    
    void on_pose_received(const turtlesim::msg::Pose::SharedPtr pose) {
        // 获取当前位置信息
        auto current_x = pose->x;
        auto current_y = pose->y;
        auto current_theta = pose->theta;
        RCLCPP_INFO(get_logger(), "current pose x: %f, y: %f", current_x, current_y);

        // 计算当前位置与目标位置的距离差和角度差
        auto distance = std::sqrt(
            (target_x_ - current_x)*(target_x_ - current_x) + (target_y_ - current_y)* (target_y_ - current_y)
        );
        auto angle = std::atan2(target_y_-current_y, target_x_-current_x) - current_theta;
		
        // 控制策略
        auto msg = geometry_msgs::msg::Twist();
        if (distance > 0.1) {
            if (fabs(angle) > 0.2) {
                msg.angular.z = fabs(angle);
            } else {
                msg.linear.x = k * distance;
            }
        }
        if (msg.linear.x > max_speed_) {
            msg.linear.x = max_speed_;
        }
        publisher_->publish(msg);
    }
private:
    rclcpp::Publisher<geometry_msgs::msg::Twist>::SharedPtr publisher_;
    rclcpp::Subscription<turtlesim::msg::Pose>::SharedPtr subscriber_;
    double target_x_{10.0};
    double target_y_{10.0};
    double k{1.0};           // the factor
    double max_speed_{2.0};  
};

int main(int argc, char ** argv) {
    rclcpp::init(argc, argv);
    auto node = std::make_shared<TurtleControl>("turtle_control");
    rclcpp::spin(node);
    rclcpp::shutdown();

    return 0;
}
```

### 3.4 话题通信最佳实践

#### 3.4.1 工程架构设计

> 需求：1. 通过这个小工具可以看到系统的实时状态信息，包括记录信息的时间、主机名称、CPU使用率、内存使用率、内存总大小、剩余可用内存、网络接收量、网络发送量
>
> 2. 要有一个简单的界面显示上述信息
> 3. 要能在局域网内其它主机上查看数据

任务：1. 要能获取系统状态信息？ Python库psutils

2. 要有一个展示界面？Qt
3. 要能共享数据？ROS2话题

#### 3.4.2 自定义通信接口

##### 1. 创建功能包

创建自定义接口功能包只能用C++类型（默认构建类型）

`ros2 pkg create status_interfaces --dependencies builtin_interfaces rosidl_default_generators --license Apache-2.0` 

##### 2. 创建自定义接口文件

在功能包文件夹下创建 `msg` 文件夹，`xxx.msg` 使用大驼峰命名法

<img src="C:/Users/13489/AppData/Roaming/Typora/typora-user-images/image-20261010145555088.png" alt="image-20261010145555088" style="zoom:60%;" />

##### 3. 编写自定义接口

```python
builtin_interfaces/Time stamp   # 时间戳 （Time实际在builtin_interfaces/msg/Time，在msg目录下可省略msg）
string host_name                # 主机名称
float32 cpu_percent
float32 memory_percent
float32 memory_total
float32 memory_available
float64 net_sent
float64 net_recv 
```

##### 5. CMakeLists.txt

```cmake
find_package(builtin_interfaces REQUIRED)
find_package(rosidl_default_generators REQUIRED)

# Cmake函数 来自rosidl_default_generators，用于将msg等消息接口定义文件转换成库或者头文件
rosidl_generate_interfaces(
  ${PROJECT_NAME}   
  "msg/SystemStatus.msg"
  DEPENDENCIES builtin_interfaces    # 因为stamp依赖其它接口
)
```

##### 6. package.xml

```xml
# 声明这是包含功能接口的功能包
<member_of_group>rosidl_default_generators</member_of_group>
```

##### 检验

`colcon build` 构建之后会生成 `cpp`、``python` 版本的类

查看自定义接口

```shell
os2@ros2-virtual-machine:~/chapt3/topic_practice_ws$ ros2 interface show status_interfaces/msg/SystemStatus 
builtin_interfaces/Time stamp   # record the timestamp
        int32 sec
        uint32 nanosec
string host_name                #
float32 cpu_percent
float32 memory_percent
float32 memory_total
float32 memory_available

float64 net_sent
float64 net_recv
```

#### 3.4.3 系统信息获取与发布

##### 编写代码

导入我们自定义的消息接口/消息类型 `from status_interfaces.msg import SystemStatus` 

创建发布者: `self.create_publisher(SystemStatus: msg_type, "sys_status": 话题名字, 10: Qos)` 

创建定时器: `self.create_timer(1.0, self.time_callback: 回调函数)` 

在回调函数中定时发布消息: `self.status_publisher_.publish(msg) `

```python
import rclpy
from status_interfaces.msg import SystemStatus
from rclpy.node import Node
import psutil              # 获取cpu、memory信息
import platform            # 获取主机名

class SysStatusPub(Node):
    def __init__(self, node_name):
        super().__init__(node_name)
        self.status_publisher_ = self.create_publisher(SystemStatus, "sys_status", 10)
        self.timer_ = self.create_timer(1.0, self.time_callback)

    def time_callback(self):
        cpu_percent = psutil.cpu_percent()
        memory_info = psutil.virtual_memory()
        net_io_counters = psutil.net_io_counters()

        msg = SystemStatus()                                 # 封装msg
        msg.stamp = self.get_clock().now().to_msg()          # 获取时间戳
        msg.host_name = platform.node()
        msg.cpu_percent = cpu_percent
        msg.memory_percent = memory_info.percent 
        msg.memory_total = memory_info.total / 1024 / 1024   # 单位为 MB
        msg.memory_available = memory_info.available / 1024 / 1024
        msg.net_sent = net_io_counters.bytes_sent / 1024 / 1024
        msg.net_recv = net_io_counters.bytes_recv / 1024 / 1024

        self.get_logger().info(f"发布消息: {msg}")
        self.status_publisher_.publish(msg)

def main():
    rclpy.init()
    node = SysStatusPub("sys_status_node")
    rclpy.spin(node)
    rclpy.shutdown()
```

##### 检验

在 `setup.py` 中注册，然后 `colcon build`， 得到可执行文件，更改环境变量，运行，观察到发布消息。

新开一个终端检查话题发布的消息：

```shell
ros2@ros2-virtual-machine:~/chapt3/topic_practice_ws$ ros2 topic list -t
/parameter_events [rcl_interfaces/msg/ParameterEvent]
/rosout [rcl_interfaces/msg/Log]
/sys_status [status_interfaces/msg/SystemStatus]      # 自定义的话题
```

```shell
ros2@ros2-virtual-machine:~/chapt3/topic_practice_ws$ source install/setup.bash
ros2@ros2-virtual-machine:~/chapt3/topic_practice_ws$ ros2 topic echo sys_status
stamp:
  sec: 1791682488
  nanosec: 363392130
host_name: ros2-virtual-machine
cpu_percent: 2.299999952316284
memory_percent: 27.100000381469727
memory_total: 7894.4375
memory_available: 5757.07421875
net_sent: 2.5600624084472656
net_recv: 3.9290971755981445
---
```

#### 3.4.4 在功能包中使用Qt

关于头文件报错在`c_cpp_properties.json`文件中添加头文件路径: `"/usr/include/x86_64-linux-gnu/qt5/**"` 

```c++
#include <QApplication>
#include <QLabel>
#include <QString>

int main(int argc, char ** argv) {
    // QApplication app = QApplication();
    QApplication app(argc, argv);
    QLabel * label = new QLabel();
    QString message = QString::fromStdString("hello qt!");
    label->setText(message);
    label->show();
    app.exec();         // 执行应用，阻塞
    return 0;
}
```

**CMakeLists.txt配置**

```cmake
find_package(Qt5 REQUIRED COMPONENTS Widgets) 
add_executable(hello_qt src/hello_qt.cpp)
target_link_libraries(hello_qt Qt5::Widgets)  # Qt没有ros2提供的ament方法
```

#### 3.4.5 订阅消息并用Qt显示

##### main

创建功能包的时候添加依赖: `status_interfaces`

创建订阅者: 

```c++
create_subscription<SystemStatus>("sys_status: 话题名字（须和发布者保持一致）", 10, 
        [this](const SystemStatus::SharedPtr msg)->void {
            label_->setText(this->get_qstr_from_msg(msg));
        });  //使用lambda创建回调函数
```

因为 `spin()` 和 `exec()` 都会进行阻塞运行，所以启动线程运行`node` 

```cpp
std::thread spin_thread([node]()->void{
        rclcpp::spin(node);
    });
```

```cpp
#include <QApplication>
#include <QLabel>
#include <QString>
#include <rclcpp/rclcpp.hpp>
#include <status_interfaces/msg/system_status.hpp>  //导入自定义消息接口

using SystemStatus = status_interfaces::msg::SystemStatus;

class SysStatusDisplay : public rclcpp::Node {
public:
    SysStatusDisplay(const std::string& node_name) : rclcpp::Node(node_name) {
        label_ = new QLabel();
        subscriber_ = this->create_subscription<SystemStatus>("sys_status", 10, 
        [this](const SystemStatus::SharedPtr msg)->void {
            label_->setText(this->get_qstr_from_msg(msg));
        });
        // 如果没有发布者发布消息，则显示一个空的msg
        label_->setText(this->get_qstr_from_msg(std::make_shared<SystemStatus>()));
        label_->show();
    }
    QString get_qstr_from_msg(const SystemStatus::SharedPtr msg) {
        std::stringstream ss;
        ss << "========系统状态信息可视化工具=========\n" <<
        "数 据 时 间:\t" << msg->stamp.sec << "\ts\n" <<
        "主 机 名 称:\t" << msg->host_name << "\t\n" <<
        "CPU 使 用 率:\t" << msg->cpu_percent << "\t%\n" << 
        "内 存 使 用 率:\t" << msg->memory_percent << "\t%\n" <<
        "内 存 总 量:\t" << msg->memory_total << "\tMB\n" << 
        "剩 余 可 用 内 存:\t" << msg->memory_available << "\tMB\n" << 
        "网 络 发 送 量:\t" << msg->net_sent << "\tMB\n" << 
        "网 络 接 收 量:\t" << msg->net_recv << "\tMB\n" <<
        "====================================";
        return QString::fromStdString(ss.str());
    }
private:
    rclcpp::Subscription<SystemStatus>::SharedPtr subscriber_;
    QLabel * label_;
};

int main(int argc, char ** argv) {
    rclcpp::init(argc, argv);
    QApplication app(argc, argv);
    auto node = std::make_shared<SysStatusDisplay>("sys_status_display_node");
    std::thread spin_thread([node]()->void{
        rclcpp::spin(node);
    });
    
    app.exec();           // 执行应用，阻塞
    rclcpp::shutdown();
    return 0;
}
```

##### CMakeLists.txt

```cmake
add_executable(sys_status_display src/sys_status_display.cpp)
target_link_libraries(sys_status_display Qt5::Widgets)   # 链接qt相关的库
ament_target_dependencies(sys_status_display rclcpp status_interfaces)
```

可以看到运行后显示为空：<img src="C:/Users/13489/AppData/Roaming/Typora/typora-user-images/image-20261011114801670.png" alt="image-20261011114801670" style="zoom:50%;"/>



运行发布者节点后：<img src="C:/Users/13489/AppData/Roaming/Typora/typora-user-images/image-20261011114907861.png" alt="image-20261011114907861" style="zoom:50%;" />

## 4. 服务

### 4.1 服务与参数

#### 4.1.1服务通信介绍

`ros2 service list -t ` 列出当前所有服务以及对应的消息接口

```shell
ros2@ros2-virtual-machine:~$ ros2 service list -t
/clear [std_srvs/srv/Empty]         # 服务名字  服务消息接口
/kill [turtlesim/srv/Kill]
/reset [std_srvs/srv/Empty]
/spawn [turtlesim/srv/Spawn]
/turtle1/set_pen [turtlesim/srv/SetPen]
···
```

`ros2 interface show 服务消息接口 ` 查看 `turtlesim/srv/Spawn ` 消息接口

上半部分为请求参数
下半部分为返回值

```shell
ros2@ros2-virtual-machine:~$ ros2 interface show turtlesim/srv/Spawn
float32 x
float32 y
float32 theta
string name # Optional.  A unique name will be created and returned if this is empty
---
string name
```

请求服务 `ros2 service call /服务名字 服务消息接口 参数`

```shell
ros2@ros2-virtual-machine:~$ ros2 service call /spawn turtlesim/srv/Spawn "{x: 1, y: 1}"
```

使用界面请求服务:

<img src="C:/Users/13489/AppData/Roaming/Typora/typora-user-images/image-20261012211931357.png" alt="image-20261012211931357" style="zoom:50%;" />

#### 4.1.2 基于服务的参数通信

```shell
ros2@ros2-virtual-machine:~$ ros2 param list
/turtlesim:
  background_b
  background_g
  background_r
  qos_overrides./parameter_events.publisher.depth
  qos_overrides./parameter_events.publisher.durability
  qos_overrides./parameter_events.publisher.history
  qos_overrides./parameter_events.publisher.reliability
  use_sim_time
```

输出显示 `turtlesim` 节点正在运行，并且列出了它所有的参数

**查看某个参数的值：**

```shell
ros2@ros2-virtual-machine:~$ ros2 param get turtlesim background_r
Integer value is: 69
```



**查看参数类型和描述：**

```shell
ros2@ros2-virtual-machine:~$ ros2 param describe turtlesim background_r
Parameter name: background_r
  Type: integer
  Description: Red channel of the background color
  Constraints:
    Min value: 0
    Max value: 255
    Step: 1
```

**修改参数**

```shell
ros2@ros2-virtual-machine:~$ ros2 param set turtlesim background_r 200
Set parameter successful
```

**导出所有参数到文件：**

```shell
ros2@ros2-virtual-machine:~$ ros2 param dump turtlesim > turtlesim_param.yaml
ros2@ros2-virtual-machine:~$ cat turtlesim_param.yaml 
/turtlesim:
  ros__parameters:
    background_b: 255
    background_g: 86
    background_r: 200
    qos_overrides:
      /parameter_events:
        publisher:
          depth: 1000
          durability: volatile
          history: keep_last
          reliability: reliable
    use_sim_time: false
```

### 4.2 Python服务通信-人脸检测

> 需求：创建一个人脸检测服务，提供图像，返回人脸数量位置信息
>
> 难点分析：
> 1.人脸怎么识别？ `face_recognition`
> 2.图片数据和结果怎么传递？服务通信
> 3.没有合适的消息接口？自定义一个

#### 4.2.1 自定义服务接口

1.创建功能包: `ros2 pkg create chapt4_interface --dependencies rosdil_default_generators sensor_msgs --license Apache-2.0`  

2.创建服务接口: 创建 `srv` 文件夹，创建 `FaceDetector.srv`

```python
sensor_msgs/Image image    # 人脸图像
---
int16 number              
float32 use_time
int32[] top
int32[] bottom
int32[] left
int32[] right
```

3.`CMakeLists.txt` 中 

```cmake
rosidl_generate_interfaces(
  ${PROJECT_NAME}   
  "srv/FaceDetector.srv"
  DEPENDENCIES sensor_msgs    # 因为image依赖其它接口
)
```

4.`package.xml` 中

```xml
# 声明这是包含服务功能接口的功能包
<member_of_group>rosidl_interface_packages</member_of_group>
```

5.编译、更改环境变量、 `ros2 interface show chapt4_interface/srv/FaceDetector` 

```shell
ros2@ros2-virtual-machine:~/chapt4/chapt4_ws$ ros2 interface show chapt4_interfaces/srv/FaceDetector
sensor_msgs/Image image  # 人脸图像
        std_msgs/Header header #
                builtin_interfaces/Time stamp
                        int32 sec
                        uint32 nanosec
                string frame_id
        string format                #
        uint8[] data                 #
---
int16 number             # 人脸数量
float32 use_time         # 识别耗时
int32[] top
int32[] bottom
int32[] left
int32[] right
```

#### 4.2.2 人脸检测

下载人脸检测包: `pip3 install face_recognition -i https://pypi.tuna.tsinghua.edu.cn/simple`

创建功能包：`ros2 pkg create demo_python_service --dependencies rclpy chapt4_interfaces --license Apache-2.0`

**`resource` 目录下的图片不会拷贝到 `install` 目录下**，所以需要在 `setup.py` 中手动设置

```python
data_files=[
        ···
        ('share/' + package_name + '/resource', ['resource/default.jpg']), # 目标路径 图片路径
    ],
```

```python
import face_recognition
import cv2
from ament_index_python.packages import get_package_share_directory  # 获取功能包share目录的绝对路径

def main():
    # 获取图片绝对路径
    default_image_path = get_package_share_directory("demo_python_service") + "/resource/default.jpg"
    # 使用cv2加载图片
    image = cv2.imread(default_image_path)
    # 检测人脸
    face_locations = face_recognition.face_locations(image, number_of_times_to_upsample=1, model="hog")
    # 绘制人脸框
    for top,right,bottom,left in face_locations:
        cv2.rectangle(image, (left,top), (right,bottom), (255,0,0), 4)
    #结果显示
    cv2.imshow("face show result", image)
    cv2.waitKey(0)
```

#### 4.2.3 人脸检测服务实现

图片的两种格式：

`ROS2:sensor_msgs   <==CvBridge==>  OpenCV ` 

> 服务实现步骤：
> 1.创建服务，接收请求Request
> 2.调用face_recognition识别
> 3.处理识别结果合成Response返回

创建服务: `create_service(FaceDetector:消息类型, "face_detect":服务名字, self.face_detect_callback: 回调函数)`

```python
from chapt4_interfaces.srv import FaceDetector
import rclpy
from rclpy.node import Node
import face_recognition
import cv2
from ament_index_python.packages import get_package_share_directory  
from cv_bridge import CvBridge
import time

class FaceDetectNode(Node):
    def __init__(self, node_name):
        super().__init__(node_name)
        self.service = self.create_service(FaceDetector, "face_detect", self.face_detect_callback)
        self.bridge = CvBridge()
        self.number_of_times_to_upsamp = 1
        self.model = "hog"
        self.default_image_path = get_package_share_directory("demo_python_service") + "/resource/default.jpg"
        self.get_logger().info(f"{node_name} 人脸检测服务已经启动！")
    
    def face_detect_callback(self, request, response):
        if request.image.data:     # 判断是否有图片
            cv2_image = self.bridge.imgmsg_to_cv2(request.image)
        else:
            cv2_image = cv2.imread(self.default_image_path)
            self.get_logger().info("传入图像为空，使用默认图像!")
        start = time.time()
        self.get_logger().info("开始人脸检测...")
        face_locations = face_recognition.face_locations(cv2_image, 
                        number_of_times_to_upsample=self.number_of_times_to_upsamp, model=self.model)
        response.use_time = time.time() - start
        self.get_logger().info("人脸检测完毕！")
        response.number = len(face_locations)
        for top,right,bottom,left in face_locations:
            response.top.append(top)
            response.right.append(right)
            response.bottom.append(bottom)
            response.left.append(left)
        return response

def main():
    rclpy.init()
    node = FaceDetectNode("face_detect_node")
    rclpy.spin(node)
    rclpy.shutdown()
```

#### 4.2.4 人脸检测客户端

> 客户端实现步骤：
> 1.创建服务客户端
> 2.构建Request，发送请求
> 3.处理返回的Response，绘制人脸显示框

创建服务客户端: `create_client(FaceDetector, "face_detect")` 

发送请求: `future = self.client.call_async(request)` 

等待服务返回结果: `rclpy.spin_until_future_complete(self, future)` 

```python
from chapt4_interfaces.srv import FaceDetector
import rclpy
from rclpy.node import Node
import face_recognition
import cv2
from ament_index_python.packages import get_package_share_directory  
from cv_bridge import CvBridge
import time

class FaceDetectClientNode(Node):
    def __init__(self, node_name):
        super().__init__(node_name)
        self.client = self.create_client(FaceDetector, "face_detect")
        self.bridge = CvBridge()
        self.default_image_path = get_package_share_directory("demo_python_service") + "/resource/test1.jpg"
        self.cv2_image = cv2.imread(self.default_image_path)
    
    def sent_request(self):
        # 1.判断服务是否在线
        while not self.client.wait_for_service(timeout_sec=1):
           self.get_logger().info("等待服务上线！")
        # 2. 构造request
        request = FaceDetector.Request()
        request.image = self.bridge.cv2_to_imgmsg(self.cv2_image)
        # 3.发送请求并等待完成
        future = self.client.call_async(request)
        # while future.done() is False:
        #     time.sleep(1)   # 阻塞，而ros2是单线程，future等待结果，而线程阻塞
        rclpy.spin_until_future_complete(self, future)   # 等待服务返回结果
        response = future.result()
        self.get_logger().info(f"人脸检测完毕！共有 {response.number} 张人脸")
        self.show_response(response)

    def show_response(self, response):
        for i in range(response.number):
            top = response.top[i]
            left = response.left[i]
            bottom = response.bottom[i]
            right = response.right[i]

            cv2.rectangle(self.cv2_image,(left,top),(right,bottom),(255,0,0),4)

        cv2.imshow("face show result", self.cv2_image)
        cv2.waitKey(0)   # 也是阻塞，导致spin无法运行（但我们只发送一次图像)
          

def main():
    rclpy.init()
    node = FaceDetectClientNode("face_detect_client_node")
    node.sent_request()
    rclpy.spin(node)
    rclpy.shutdown()
```

### 4.3 C++服务通信-巡逻海龟

> 需求：让小海龟在海龟模拟器中随机游走进行巡逻
>
> 分析：
> 1.控制小海龟到目标点：通过位置接口、速度接口实现，怎么改成动态接收？使用服务
> 2.用什么接口？自定义一个
> 3.随机游走？客户端来产生随机点，请求巡逻服务
>
> <img src="C:/Users/13489/AppData/Roaming/Typora/typora-user-images/image-20261014195125093.png" alt="image-20261014195125093" style="zoom:50%;" />

#### 4.3.1 自定义服务接口

在 `srv` 中新建 `Patrol.srv` :

```python
float32 target_x
float32 target_y
---
int8 SUCCESS = 1
int8 FAIL = 0
int8 result   #结果，SUCCESS/FAIL取其一
```

#### 4.3.2 服务端代码实现

在 `3.3.2` 节的代码基础上创建服务: `create_service<Patrol>("patrol": 服务名字,回调函数)` 回调函数的参数为 `request、response` 

```cpp
service_ = this->create_service<Patrol>("patrol", 
            [this](const Patrol::Request::SharedPtr request, Patrol::Response::SharedPtr response)-> void{
                if (
                    (0 < request->target_x && request->target_x < 12.0f) && 
                    (0 < request->target_y && request->target_y < 12.0f)
                ) {
                    this->target_x_ = request->target_x;
                    this->target_y_ = request->target_y;
                    response->result = Patrol::Response::SUCCESS;
                } else {
                    response->result = Patrol::Response::FAIL;
                }
        });
    }
```

#### 4.3.3 客户端代码实现

创建客户端: `create_client<Patrol>("patrol")`

创建定时器，使用 `ctime` 库生成随机点，通过回调函数发送请求: `async_send_request(request, [](rclcpp::Client<Patrol>::SharedFuture future_result){};)` ，得到相应自动调用 `lambda`函数

```cpp
#include "rclcpp/rclcpp.hpp"
#include "chapt4_interfaces/srv/patrol.hpp"
#include <chrono>
#include <ctime>

using Patrol = chapt4_interfaces::srv::Patrol;
using namespace std::chrono_literals;  // 可以在创建定时器时直接传入1000ms

class PatrolClient : public rclcpp::Node {
public:
    PatrolClient(const std::string& node_name) : Node(node_name) {
        srand(time(nullptr));         // 初始化随机种子
        client_ = this->create_client<Patrol>("patrol");
        timer_ = this->create_wall_timer(10s, [this]() -> void {
            // 1.判断服务上线
            while (!this->client_->wait_for_service(1s)) {
                if (!rclcpp::ok()) {
                    RCLCPP_ERROR(get_logger(), "等待服务上线过程中挂了<_>");
                    return;
                }
                RCLCPP_INFO(get_logger(), "等待服务上线中...");
            }
            // 2.构造Request
            // Patrol::Request::SharedPtr request;
            auto request = std::make_shared<Patrol::Request>();
            request->target_x = rand() % 15;
            request->target_y = rand() % 15;
            RCLCPP_INFO(get_logger(), "产生随机点: %f, %f", request->target_x, request->target_y);
            // 3.发送请求并等待完成
            client_->async_send_request(request, [this](rclcpp::Client<Patrol>::SharedFuture future_result) -> void{
                auto response = future_result.get();
                if (response->result == Patrol::Response::SUCCESS) {
                    RCLCPP_INFO(get_logger(), "请求巡逻服务成功！");
                }
                if (response->result == Patrol::Response::FAIL) {
                    RCLCPP_INFO(get_logger(), "请求巡逻服务失败<_>");
                }
            });
        });
    }
    
private:
    rclcpp::Client<Patrol>::SharedPtr client_;
    rclcpp::TimerBase::SharedPtr timer_; 
};
```

### 4.4 在Python节点使用参数

#### 4.4.1 参数声明设置

```python
self.declare_parameter('number_of_times_to_upsamp', 1)   # 声明参数
self.declare_parameter('model', 'hog')                   # 声明参数
self.number_of_times_to_upsamp = self.get_parameter('number_of_times_to_upsamp').value
self.model = self.get_parameter('model').value
```

获取节点及对应参数: `ros2 param list`

```shell
/face_detect_node:
  model
  number_of_times_to_upsamp
  use_sim_time`
```

设置参数：

```shell
ros2@ros2-virtual-machine:~/chapt4/chapt4_ws$ ros2 param set face_detect_node number_of_times_to_upsamp 2
Set parameter successful
```

获取参数的值：

```shell
ros2@ros2-virtual-machine:~/chapt4/chapt4_ws$ ros2 param get face_detect_node number_of_times_to_upsamp
Integer value is: 2
```

带参数启动节点：

```shell
ros2@ros2-virtual-machine:~/chapt4/chapt4_ws$ ros2 run demo_python_service face_detect_node --ros-args -p model:=cnn
```

#### 4.4.2 订阅参数更新

```python
# 注册回调：参数被 set 时触发
self.add_on_set_parameters_callback(self.parameters_callback)
```

在进行设置参数时执行回调函数：

```shell
ros2@ros2-virtual-machine:~/chapt4/chapt4_ws$ ros2 param set face_detect_node number_of_times_to_upsamp 2
[INFO] [1792069010.431117042] [face_detect_node]: model: hog
```

### 4.5 在C++节点使用参数

#### 4.5.1 C++参数声明与设置

> 将4.3节的巡逻海龟项目服务端中控制器的比例系数 `k` 和最大速度 `max_speed` 进行参数化

```c++
this->declare_parameter("k", 1.0);
this->declare_parameter("max_speed", 2.0);
this->get_parameter("k", k_);
this->get_parameter("max_speed", max_speed_);
```

#### 4.5.2 C++接收参数事件

> 对SetParameter服务添加回调函数：注册参数校验回调，用于拦截/校验，不主动改参数

```c++
this->set_parameter_callback_handle_ = add_on_set_parameters_callback(
            [this](const std::vector<rclcpp::Parameter> & parameters){
                rcl_interfaces::msg::SetParametersResult result;
                result.successful = true;
                for (const auto & parameter : parameters) {
                    RCLCPP_INFO(get_logger(), "参数更新%s = %f", parameter.get_name().c_str(), parameter.as_double());
                    if (parameter.get_name() == "k") {
                        this->k_ = parameter.as_double();
                    } else if (parameter.get_name() == "max_speed") {
                        this->max_speed_ = parameter.as_double();
                    }
                }
                return result;
            });

OnSetParametersCallbackHandle::SharedPtr set_parameter_callback_handle_;
```

#### 4.5.3 通过服务客户端发送请求修改参数

```c++
// 创建设置参数客户端
#include "rcl_interfaces/msg/parameter.hpp"
#include "rcl_interfaces/msg/parameter_value.hpp"
#include "rcl_interfaces/msg/set_parameters_result.hpp"
#include "rcl_interfaces/srv/set_parameters.hpp"

using SetParameters = rcl_interfaces::srv::SetParameters;

// 创建客户端发送请求，返回结果
SetParameters::Response::SharedPtr call_set_parameter(const rcl_interfaces::msg::Parameter & param) {
    auto set_param_client = this->create_client<SetParameters>("/turtle_control/set_parameters");
    // 1.判断服务上线
    while (!set_param_client->wait_for_service(1s)) {
        if (!rclcpp::ok()) {
            RCLCPP_ERROR(get_logger(), "等待服务上线过程中挂了<_>");
            return nullptr;
        }
        RCLCPP_INFO(get_logger(), "等待服务上线中...");
    }
    // 2.构造Request
    auto request = std::make_shared<SetParameters::Request>();
    request->parameters.push_back(param);
    // 3.发送请求并等待完成
    auto future = set_param_client->async_send_request(request);
    rclcpp::spin_until_future_complete(this->get_node_base_interface(), future);
    auto response = future.get();
    return response;
}

// 更新参数k
void update_param_k(double k) {
    // auto setParameters = SetParameters();
    //1. 创建参数对象
    auto param = rcl_interfaces::msg::Parameter();
    param.name = "k";
    // 2. 设置参数值
    auto param_value = rcl_interfaces::msg::ParameterValue();
    param_value.type = rcl_interfaces::msg::ParameterType::PARAMETER_DOUBLE;
    param_value.double_value = k;

    param.value = param_value;
    //3. 请求更新参数并处理
    auto response = call_set_parameter(param);
    if (response == nullptr) {
        RCLCPP_INFO(get_logger(), "参数更新失败");
        return;
    }
    for (auto result : response->results) {
        if (result.successful == false) {
            RCLCPP_INFO(get_logger(), "参数更新失败, 原因: %s", result.reason.c_str());
            return;
        } else {
            RCLCPP_INFO(get_logger(), "参数更新成功！");
        }
    }
}
```

### 4.6 使用launch启动脚本

#### 4.6.1 使用launch启动多个节点

> 前面章节启动多个节点需要开多个终端，每个终端更改环境变量然后启动节点

新建 `launch` 文件夹，新建 `demo.launch.py` 文件

<img src="C:/Users/13489/AppData/Roaming/Typora/typora-user-images/image-20261021130252202.png" alt="image-20261021130252202" style="zoom:50%;" />

```python
import launch
import launch_ros

def generate_launch_description():
    action_node_turtlesim_node = launch_ros.actions.Node(
        package="turtlesim",
        executable="turtlesim_node",
        output = "screen"
    )
    action_node_patrol_client = launch_ros.actions.Node(
        package="demo_cpp_service",
        executable="patrol_client",
        output = "log"
    )
    action_node_turtle_control = launch_ros.actions.Node(
        package="demo_cpp_service",
        executable="turtle_control",
        output = "both"
    )
    return launch.LaunchDescription([
        # action动作
        action_node_turtlesim_node,
        action_node_patrol_client,
        action_node_turtle_control
    ])
```

**拷贝到 `install` 目录**

```cmake
install(DIRECTORY launch
  DESTINATION share/${PROJECT_NAME}
)
```

```python
from glob import glob
data_files=[
        ('share/ament_index/resource_index/packages',
            ['resource/' + package_name]),
        ('share/' + package_name, ['package.xml']),
        ('share/' + package_name + '/launch', glob('launch/*.launch.py'))
],
# 源码目录 launch/ 下所有 *.launch.py 文件，拷贝到 ROS 2 的 share/<package_name>/launch/ 目录
```

**使用 `launch` 运行**

```shell
ros2@ros2-virtual-machine:~/chapt4/chapt4_ws$ ros2 launch demo_python_service demo.launch.py
```

#### 4.6.2 使用launch传递参数

> 通过ros2节点启动可以在后面跟上参数，但launch默认不能跟参数启动，因为launch中有多个节点，不知道将参数传递给哪个节点，需要进行配置

```python
# 1.声明一个launch参数，也需要添加action到返回值中
action_declare_arg_background_g = launch.actions.DeclareLaunchArgument("launch_arg_bg", default_value="150")
# 将参数传递给turtlesim_node节点
action_node_turtlesim_node = launch_ros.actions.Node(
        package="turtlesim",
        executable="turtlesim_node",
        output = "screen",
        # 2.把launch参数手动传递给节点，将launch参数值替换background_g
        parameters=[{'background_g': launch.substitutions.LaunchConfiguration('launch_arg_bg',default="150")}],
    )
```

启动：

```shell
ros2@ros2-virtual-machine:~/chapt4/chapt4_ws$ ros2 launch demo_cpp_service demo.launch.py launch_arg_bg:=0 # 修改参数，不带参数则使用默认值
```

#### 4.6.3 launch使用进阶

```python
import launch
import launch.launch_description_sources
import launch_ros
import os
from ament_index_python import get_package_share_directory

def generate_launch_description():
    # 动作1: 启动其他launch
    turtlesim_launch_path = os.path.join(
        get_package_share_directory("turtlesim"),
        "launch",
        "multisim.launch.py",
    )
    action_include_launch = launch.actions.IncludeLaunchDescription(
        launch.launch_description_sources.PythonLaunchDescriptionSource(
            turtlesim_launch_path
        )
    )

    # 动作2：打印
    action_log_info = launch.actions.LogInfo(msg=turtlesim_launch_path)

    # 动作3：执行进程，其实就是执行一个命令行
    action_topic_list = launch.actions.ExecuteProcess(
        cmd=['ros2', 'topic', 'list'],
        output = "screen",
    )

    # 动作4：多个动作组织成组，把多个动作放在一组
    action_group = launch.actions.GroupAction([
        # 动作5：定时器
        launch.actions.TimerAction(period=2.0, actions=[action_include_launch]),
        launch.actions.TimerAction(period=4.0, actions=[action_topic_list]),
    ])

    return launch.LaunchDescription([
        # action动作
        action_log_info,
        action_group
    ])
```

通过参数控制是否启动某个 `action`

```python
# 声明参数startup_rqt
action_declare_arg_startup_rqt = launch.actions.DeclareLaunchArgument("startup_rqt", default_value="False")
# 取值占位符：一个"将来会被替换成实际值"的对象
startup_rqt = launch.substitutions.LaunchConfiguration('startup_rqt',default="False")
# 条件动作：
action_rqt = launch.actions.ExecuteProcess(
    condition = launch.conditions.IfCondition(startup_rqt),
    cmd = ['rqt'],
)
```

运行：

```shell
ros2@ros2-virtual-machine:~/chapt4/chapt4_ws$ ros2 launch demo_cpp_service actions.launch.py startup_rqt:=True
```

## 5. 工具

### 5.1 坐标变换工具

#### 5.1.1 通过命令行使用TF

在 ROS2 的 TF 系统中，**旋转变换统一用四元数（Quaternion）表示**。

```shell
# base_laser（激光雷达）坐标系，位于 base_link（机器人本体）坐标系的正前方 0.1 米、正上方 0.2 米处，且两者朝向完全一致（无旋转）
ros2@ros2-virtual-machine:~$ ros2 run tf2_ros static_transform_publisher --x 0.1 --y 0.0 --z 0.2 --roll 0.0 --pitch 0.0 --yaw 0.0 --frame-id base_link --child-frame-id base_laser
#  wall_point 的坐标系（可以理解为一个被标记的墙壁点），它相对于 base_laser 坐标系的位置是：正前方 0.3 米处
ros2@ros2-virtual-machine:~$ ros2 run tf2_ros static_transform_publisher --x 0.3 --y 0.0 --z 0.0 --roll 0.0 --pitch 0.0 --yaw 0.0 --frame-id base_laser --child-frame-id wall_point
```

```shell
# wall_point 这个点，在 base_link 坐标系下是什么位置和姿态
ros2@ros2-virtual-machine:~$ ros2 run tf2_ros tf2_echo base_link wall_point
At time 0.0
- Translation: [0.400, 0.000, 0.200]
- Rotation: in Quaternion (xyzw) [0.000, 0.000, 0.000, 1.000]
- Rotation: in RPY (radian) [0.000, -0.000, 0.000]
- Rotation: in RPY (degree) [0.000, -0.000, 0.000]
- Matrix:
  1.000  0.000  0.000  0.400
  0.000  1.000  0.000  0.000
  0.000  0.000  1.000  0.200
  0.000  0.000  0.000  1.000
```

#### 5.1.2 TF简单原理

```shell
ros2@ros2-virtual-machine:~$ ros2 topic info /tf_static
Type: tf2_msgs/msg/TFMessage
Publisher count: 2
Subscription count: 0
ros2@ros2-virtual-machine:~$ ros2 interface show tf2_msgs/msg/TFMessage 
geometry_msgs/TransformStamped[] transforms
	std_msgs/Header header
		builtin_interfaces/Time stamp
			int32 sec
			uint32 nanosec
		string frame_id
	string child_frame_id
	Transform transform
		Vector3 translation
			float64 x
			float64 y
			float64 z
		Quaternion rotation
			float64 x 0
			float64 y 0
			float64 z 0
			float64 w 1
```

```tex
TFMessage
└── geometry_msgs/TransformStamped[] transforms   ← 一个数组
    ├── header
    │   ├── stamp (sec, nanosec)   ← 时间戳
    │   └── frame_id               ← 父坐标系
    ├── child_frame_id             ← 子坐标系
    └── transform
        ├── translation (x,y,z)    ← 平移
        └── rotation (x,y,z,w)     ← 四元数旋转
```

```shell
ros2@ros2-virtual-machine:~$ ros2 topic echo /tf_static
transforms:
- header:
    stamp:
      sec: 1792581909
      nanosec: 165207103
    frame_id: base_link
  child_frame_id: base_laser
  transform:
    translation:
      x: 0.1
      y: 0.0
      z: 0.2
    rotation:
      x: 0.0
      y: 0.0
      z: 0.0
      w: 1.0
---
transforms:
- header:
    stamp:
      sec: 1792596524
      nanosec: 855700353
    frame_id: base_laser
  child_frame_id: wall-point
  transform:
    translation:
      x: 0.3
      y: 0.0
      z: 0.0
    rotation:
      x: 0.0
      y: 0.0
      z: 0.0
      w: 1.0
---
```

### 5.2 Python坐标变换

#### 5.2.1 发布静态TF

> <img src="C:/Users/13489/AppData/Roaming/Typora/typora-user-images/image-20261022223720201.png" alt="image-20261022223720201" style="zoom:40%;" />

通过发布 `/tf_static` 话题来实现

前面通过 `ros2 interface show tf2_msgs/msg/TFMessage` 查看消息接口类型为 `geometry_msgs/mag/TransformStamped`

`math.radians(180)`: 将度数转换成弧度制

```python
import rclpy
from rclpy.node import Node
from tf2_ros import StaticTransformBroadcaster  # 静态坐标发布器
from geometry_msgs.msg import TransformStamped  # 消息接口
from tf_transformations import quaternion_from_euler  # 欧拉角转换四元数
import math

class StaticTFBroadcaster(Node):
    def __init__(self, node_name):
        super().__init__(node_name)
        self.static_broadcaster_ = StaticTransformBroadcaster(self)
        self.publish_static_tf()

    def publish_static_tf(self):
        # 发布静态TF，从 base_link 到 camera_link 的坐标关系
        # 这个 transform 描述的是 camera_link 在 base_link 坐标系下的位姿
        tranform = TransformStamped()
        tranform.header.frame_id = 'base_link'
        tranform.header.stamp = self.get_clock().now().to_msg()
        tranform.child_frame_id = 'camera_link'
		
        # 表示 camera_link 的原点在 base_link 坐标系下的位置是 (0.5, 0.3, 0.6) 米
        tranform.transform.translation.x = 0.5
        tranform.transform.translation.y = 0.3
        tranform.transform.translation.z = 0.6

        # q = (x, y, z, w)
        # camera_link 相对于 base_link 绕 X 轴旋转 180°(roll = 180°，pitch = 0，yaw = 0)
        q = quaternion_from_euler(math.radians(180), 0, 0)
        tranform.transform.rotation.x = q[0]
        tranform.transform.rotation.y = q[1]
        tranform.transform.rotation.z = q[2]
        tranform.transform.rotation.w = q[3]

        self.static_broadcaster_.sendTransform(tranform)
        self.get_logger().info(f"发布静态TF：{tranform}")
        
def main():
    rclpy.init()
    node = StaticTFBroadcaster("static_tf_broadcaster_node")
    rclpy.spin(node)
    rclpy.shutdown()
```

```shell
ros2@ros2-virtual-machine:~/chapt5/chapt5_ws/src/demo_python_tf/demo_python_tf$ ros2 topic echo /tf_static
transforms:
- header:
    stamp:
      sec: 1792679332
      nanosec: 963473011
    frame_id: base_link
  child_frame_id: camera_link
  transform:
    translation:
      x: 0.5
      y: 0.3
      z: 0.6
    rotation:
      x: 1.0
      y: 0.0
      z: 0.0
      w: 6.123233995736766e-17
```

#### 5.2.2 发布动态TF

将静态坐标发布器 `StaticTransformBroadcaster` 改成动态的 `TransformBroadcaster`

```python
from tf2_ros import TransformBroadcaster  # 动态坐标发布器
...
def __init__():
    ...
	self.broadcaster_ = TransformBroadcaster(self)
    self.timer_ = self.create_timer(0.01, self.publish_tf)
```

#### 5.2.3 查询TF关系

```python
from tf2_ros import TransformListener, Buffer

def __init__(self, node_name):
        super().__init__(node_name)
        self.buffer_ = Buffer()
        self.listener_ = TransformListener(self.buffer_, self)
        self.timer_ = self.create_timer(1, self.get_transform)

    def get_transform(self):
        # 查询坐标关系 buffer_
        try:
            result = self.buffer_.lookup_transform('base_link', 'bottle_link', 
                    rclpy.time.Time(seconds=0), rclpy.time.Duration(seconds=1))
            transform = result.transform
            self.get_logger().info(f"平移: {transform.translation}")
            self.get_logger().info(f"旋转: {transform.rotation}")
            rotation_euler = euler_from_quaternion([
                transform.rotation.x,
                transform.rotation.y,
                transform.rotation.z,
                transform.rotation.w,
            ])
            self.get_logger().info(f"旋转RPY: {rotation_euler}")
        except Exception as e:
            self.get_logger().warn(f"查询坐标关系失败，原因: {str(e)}")
```

### 5.3 C++坐标变换

> 需求：
> <img src="C:/Users/13489/AppData/Roaming/Typora/typora-user-images/image-20261024103404951.png" alt="image-20261024103404951" style="zoom:40%;" />

#### 5.3.1 发布静态坐标

```c++
#include "geometry_msgs/msg/transform_stamped.hpp"  // 消息接口类型
#include "rclcpp/rclcpp.hpp"
#include "tf2/LinearMath/Quaternion.hpp"            // 提供tf2::Quaternion类
#include "tf2_geometry_msgs/tf2_geometry_msgs.hpp"  // 四元数转消息
#include "tf2_ros/static_transform_broadcaster.hpp"

class StaticTFBroadcaster : public rclcpp::Node {
public:
    StaticTFBroadcaster(const std::string& node_name) : Node(node_name) {
        broadcaster_ = std::make_shared<tf2_ros::StaticTransformBroadcaster>(this);
        this->publish_tf();
    }
    void publish_tf() {
        geometry_msgs::msg::TransformStamped transform;
        transform.header.frame_id = "map";
        transform.header.stamp = this->get_clock()->now();
        transform.child_frame_id = "target_pointer";

        transform.transform.translation.x = 5.0;
        transform.transform.translation.y = 3.0;
        transform.transform.translation.z = 0.0;

        tf2::Quaternion q;
        q.setRPY(0.0, 0.0, 60 * M_PI / 180);      // 用 RPY(roll,pitch,yaw) 构造
        transform.transform.rotation = tf2::toMsg(q); // 转成 ROS 消息类型

        this->broadcaster_->sendTransform(transform);
    }
private:
    std::shared_ptr<tf2_ros::StaticTransformBroadcaster> broadcaster_;
};

int main(int argc, char** argv) {
    rclcpp::init(argc, argv);
    auto node = std::make_shared<StaticTFBroadcaster>("static_cpp_broadcaster_node");
    rclcpp::spin(node);
    rclcpp::shutdown();
    return 0;
}
```

#### 5.3.2 发布动态坐标

```c++
#include "geometry_msgs/msg/transform_stamped.hpp"
#include "rclcpp/rclcpp.hpp"
#include "tf2/LinearMath/Quaternion.hpp"            // 提供tf2::Quaternion类
#include "tf2_geometry_msgs/tf2_geometry_msgs.hpp"  // 四元数转消息
#include "tf2_ros/transform_broadcaster.hpp"

using namespace std::chrono_literals;

class TFBroadcaster : public rclcpp::Node {
public:
    TFBroadcaster(const std::string& node_name) : Node(node_name) {
        RCLCPP_INFO(get_logger(), node_name.c_str());
        broadcaster_ = std::make_shared<tf2_ros::TransformBroadcaster>(this);
        timer_ = this->create_wall_timer(100ms, std::bind(&TFBroadcaster::publish_tf, this));
    }
    void publish_tf() {
        geometry_msgs::msg::TransformStamped transform;
        transform.header.frame_id = "map";
        transform.header.stamp = this->get_clock()->now();
        transform.child_frame_id = "base_link";
        transform.transform.translation.x = 2.0;
        transform.transform.translation.y = 3.0;
        transform.transform.translation.z = 0.0;
        tf2::Quaternion q;
        q.setRPY(0.0, 0.0, 30 * M_PI / 180);
        transform.transform.rotation = tf2::toMsg(q);
        this->broadcaster_->sendTransform(transform);
    }
private:
    std::shared_ptr<tf2_ros::TransformBroadcaster> broadcaster_;
    rclcpp::TimerBase::SharedPtr timer_;
};

int main(int argc, char** argv) {
    rclcpp::init(argc, argv);
    auto node = std::make_shared<TFBroadcaster>("dynamic_cpp_broadcaster_node");
    rclcpp::spin(node);
    rclcpp::shutdown();
    return 0;
}
```

#### 5.3.3 查询TF关系

原理：通过订阅 `/tf` `/tf_static` 话题手机所有坐标系关系，进行计算

设置缓冲区: `buffer_ = std::make_shared<tf2_ros::Buffer>(this->get_clock());`
设置监听者: `listener_ = std::make_shared<tf2_ros::TransformListener>(*buffer_, this);`

```c++
#include "rclcpp/rclcpp.hpp"
#include "tf2_ros/transform_listener.hpp"
#include "tf2_ros/buffer.hpp"
#include "tf2/utils.hpp"       //getEulerYPR()

using namespace std::chrono_literals;

class TFListener : public rclcpp::Node {
public:
    TFListener(const std::string& node_name) : Node(node_name) {
        RCLCPP_INFO(get_logger(), node_name.c_str());
        buffer_ = std::make_shared<tf2_ros::Buffer>(this->get_clock());
        listener_ = std::make_shared<tf2_ros::TransformListener>(*buffer_, this);
        timer_ = this->create_wall_timer(1s, std::bind(&TFListener::get_transform, this));
    }
    void get_transform(){
        try {
            auto result = buffer_->lookupTransform("base_link", "target_pointer", 
                this->get_clock()->now(), rclcpp::Duration::from_seconds(1.0));
            auto translation = result.transform.translation;
            RCLCPP_INFO(get_logger(), "平移: x: %f, y: %f, z: %f", 
                translation.x, translation.y, translation.z);
            
            auto rotation = result.transform.rotation;
            double y, p, r;
            tf2::getEulerYPR(rotation, y, p, r);
            RCLCPP_INFO(get_logger(), "旋转: r: %f, p: %f, y: %f",
                y, p, r);
        }
        catch(const std::exception& e)
        {
            RCLCPP_WARN(get_logger(), "%s", e.what());
        }    
    }
private:
    std::shared_ptr<tf2_ros::TransformListener> listener_;
    rclcpp::TimerBase::SharedPtr timer_;
    std::shared_ptr<tf2_ros::Buffer> buffer_;
};

int main(int argc, char** argv) {
    rclcpp::init(argc, argv);
    auto node = std::make_shared<TFListener>("cpp_broadcaster_listener_node");
    rclcpp::spin(node);
    rclcpp::shutdown();
    return 0;
}
```

### 5.4 rqt和rviz



### 5.5 数据记录ros2 bag

```shell
# 终端 1: 启动仿真窗口
ros2 run turtlesim turtlesim_node
# 终端 2：启动键盘控制
ros2 run turtlesim turtle_teleop_key
```

此时在终端 2 中按方向键，小海龟会移动。键盘控制发出的速度指令发布在 `/turtle1/cmd_vel` 话题上。

#### 1. 记录数据

```shell
ros2 bag record /turtle1/cmd_vel
```

终端会显示正在监听 `/turtle1/cmd_vel`。此时回到键盘控制终端，控制小海龟走出一段轨迹，然后按 `Ctrl+C` 停止录制。

录制结束后，当前目录会生成一个 `rosbag2_日期-时间` 文件夹

#### 2. 播放记录

先关闭键盘控制节点（避免同时发送指令干扰回放），保持 `turtlesim_node` 运行。然后执行：

```shell
ros2 bag play rosbag2_2026_10_24-10_00_00
```

仿真窗口中的小海龟会按照你刚才键盘操作的方式重新运动一遍。

## 6. 仿真

### 6.2 使用URDF创建机器人

#### 6.2.1 帮机器人创建一个身体

在 `src` 目录下新建 `urdf` 文件夹，新建 `first_robot.urdf`

```xml
<?xml version="1.0"?>
<robot name="first_robot">
     <!-- 机器人身体部分 -->
     <link name="base_link">
        <!-- 部件的外观描述 -->
        <visual>
            <!-- 沿着自己几何中心的偏移量和旋转量 -->
            <origin xyz="0.0 0.0 0.0" rpy="0.0 0.0 0.0"/>
            <!-- 几何形状 -->
            <geometry>
                <!-- 圆柱体 半径0.1m，高度0.12m -->
                <cylinder radius="0.10" length="0.12"/>
            </geometry>
            <!-- 材质颜色 -->
            <material name="white">
                <color rgba="1.0 1.0 1.0 0.5"/>
            </material>
        </visual>
     </link>
     <!-- 机器人的IMU部件，惯性测量传感器 -->
     <link name="imu_link">
        <!-- 部件的外观描述 -->
        <visual>
            <!-- 沿着自己几何中心的偏移量和旋转量 -->
            <origin xyz="0.0 0.0 0.0" rpy="0.0 0.0 0.0"/>
            <!-- 几何形状 -->
            <geometry>
                <box size="0.02 0.02 0.02"/>
            </geometry>
            <!-- 材质颜色 -->
            <material name="black">
                <color rgba="0.0 0.0 0.0 0.5"/>
            </material>
        </visual>
     </link>
     <!-- 机器人的关节，用于组合机器人的部件 -->
     <joint name="imu_joint" type="fixed">
        <parent link="base_link"/>
        <child link="imu_link"/>
        <origin xyz="0.0 0.0 0.3" rpy="0.0 0.0 0.0"/>      
     </joint>
</robot>
```

生成 `pdf` 文件：`urdf_to_graphviz first_robot.urdf`

#### 6.2.2 在rviz中显示机器人

通过 `launch` 同时启动多个节点 `rviz2` 、`robot_state_publisher`、`joint_state_publisher` ，而 `robot_state_publisher` 需要 `robot_description`参数，数值为机器人 `urdf` 内容

```python
import launch
import launch_ros
from ament_index_python import get_package_share_directory
import os

import launch_ros.parameter_descriptions

def generate_launch_description():
    # 获取默认的urdf路径
    share_urdf_path = get_package_share_directory("fishbot_description")
    default_urdf_path = os.path.join(share_urdf_path, 'urdf', 'first_robot.urdf')
    default_rviz_config_path = os.path.join(share_urdf_path, 'config', 'display_robot_model.rviz')

    # 声明一个urdf文件路径的参数，方便修改
    action_delcare_arg_model_path = launch.actions.DeclareLaunchArgument("model", 
                    default_value=default_urdf_path, description="加载模型的默认路径")
    
    # 因为robot_state_publisher节点需要参数：机器人的描述。需要读取urdf文件内容作为参数
    robot_desc = launch_ros.parameter_descriptions.ParameterValue(
        launch.substitutions.Command(['cat ', launch.substitutions.LaunchConfiguration('model')]),
        value_type=str,
    ) 

    # 启动节点
    action_robot_state_publisher = launch_ros.actions.Node(
        name='robot_state_publisher',
        package='robot_state_publisher',
        executable='robot_state_publisher',
        parameters=[{'robot_description':robot_desc}]
    )

    action_joint_state_publisher = launch_ros.actions.Node(
        name='joint_state_publisher',
        package='joint_state_publisher',
        executable='joint_state_publisher',
    )
    action_rviz = launch_ros.actions.Node(
        name='rviz2',
        executable='rviz2',
        arguments=['-d', default_rviz_config_path],
    )

    return launch.LaunchDescription([
        action_delcare_arg_model_path,
        action_robot_state_publisher,
        action_joint_state_publisher,
        action_rviz,
    ])
```

运行后可以把 `rviz`文件保存到 `config` 文件夹下，这样每次启动可以直接打开设置好的 `rviz`文件

#### 6.2.3 使用xacro简化urdf

类似面向对象，可以利用 `<xacro:base_link length="0.12" radius="0.1"/>` 创建机器人部件

```xml
<?xml version="1.0"?>
<robot xmlns:xacro="http://www.ros.org/wiki/xacro" name="first_robot">
     <xacro:macro name = "base_link" params = "length radius">
        <!-- 机器人身体部分 -->
        <link name="base_link">
            <!-- 部件的外观描述 -->
            <visual>
                <!-- 沿着自己几何中心的偏移量和旋转量 -->
                <origin xyz="0.0 0.0 0.0" rpy="0.0 0.0 0.0"/>
                <!-- 几何形状 -->
                <geometry>
                    <!-- 圆柱体 半径0.1m，高度0.12m -->
                    <cylinder radius="${radius}" length="${length}"/>
                </geometry>
                <!-- 材质颜色 -->
                <material name="white">
                    <color rgba="1.0 1.0 1.0 0.5"/>
                </material>
            </visual>
        </link>
     </xacro:macro>

     <xacro:macro name = "imu_link" params = "name xyz">
        <!-- 机器人的IMU部件，惯性测量传感器 -->
        <link name="${name}_link">
            <!-- 部件的外观描述 -->
            <visual>
                <!-- 沿着自己几何中心的偏移量和旋转量 -->
                <origin xyz="0.0 0.0 0.0" rpy="0.0 0.0 0.0"/>
                <!-- 几何形状 -->
                <geometry>
                    <box size="0.02 0.02 0.02"/>
                </geometry>
                <!-- 材质颜色 -->
                <material name="black">
                    <color rgba="0.0 0.0 0.0 0.5"/>
                </material>
            </visual>
        </link>

        <!-- 机器人的关节，用于组合机器人的部件 -->
        <joint name="${name}_joint" type="fixed">
            <parent link="base_link"/>
            <child link="${name}_link"/>
            <origin xyz="${xyz}" rpy="0.0 0.0 0.0"/>      
        </joint>
     </xacro:macro>

     <xacro:base_link length="0.12" radius="0.1"/>
     <xacro:imu_link name="imu_up" xyz="0.0 0.0 0.03"/>
     <xacro:imu_link name="imu_down" xyz="0.0 0.0 -0.03"/>

</robot>
```

运行：`xacro 文件路径` 即可终端返回 `urdf`，因此在 `launch` 启动文件中将 `cat` 命令修改成 `xacro`即可，运行时传入参数 `model:=xacro文件路径`

#### 6.2.4 创建机器人机器传感器部件

创建 `fishbot` 文件夹，创建 `sensor` 文件夹

<img src="C:/Users/13489/AppData/Roaming/Typora/typora-user-images/image-20261028145227882.png" alt="image-20261028145227882" style="zoom:50%;" />

例如 `base.urdf.xacro` 

```xml
<?xml version="1.0"?>
<robot xmlns:xacro="http://www.ros.org/wiki/xacro">
     <xacro:macro name = "base_xacro" params = "length radius">
        <!-- 机器人身体部分 -->
        <link name="base_link">
            <!-- 部件的外观描述 -->
            <visual>
                <!-- 沿着自己几何中心的偏移量和旋转量 -->
                <origin xyz="0.0 0.0 0.0" rpy="0.0 0.0 0.0"/>
                <!-- 几何形状 -->
                <geometry>
                    <!-- 圆柱体 半径0.1m，高度0.12m -->
                    <cylinder radius="${radius}" length="${length}"/>
                </geometry>
                <!-- 材质颜色 -->
                <material name="white">
                    <color rgba="1.0 1.0 1.0 0.5"/>
                </material>
            </visual>
        </link>
     </xacro:macro>
</robot>
```

在 `fishbot.urdf.xacro` 中使用各种 `xacro` 

```xml
<?xml version="1.0"?>
<robot xmlns:xacro="http://www.ros.org/wiki/xacro" name="fishbot">
<!-- 包含传感器文件 -->
<xacro:include filename="$(find fishbot_description)/urdf/fishbot/base.urdf.xacro"/>
<xacro:include filename="$(find fishbot_description)/urdf/fishbot/sensor/camera.urdf.xacro"/>
<xacro:include filename="$(find fishbot_description)/urdf/fishbot/sensor/imu.urdf.xacro"/>
<xacro:include filename="$(find fishbot_description)/urdf/fishbot/sensor/laser.urdf.xacro"/>

<xacro:base_xacro length="0.12" radius="0.10" />
<xacro:imu_xacro xyz="0.0 0.0 0.02" />
<xacro:camera_xacro xyz="0.10 0.0 0.075" />
<xacro:laser_xacro xyz="0.0 0.0 0.10" />
</robot>
```

使用 `xacro fishbot.urdf.xacro路径` 验证是否能够生成 `urdf` 
运行效果：

<img src="C:/Users/13489/AppData/Roaming/Typora/typora-user-images/image-20261028145613364.png" alt="image-20261028145613364" style="zoom:50%;" />

#### 6.2.5 添加执行器部件

新建 `actuator` 文件夹，创建 `wheel.urdf.xacro` 轮子部件

`<origin xyz="0.0 0.0 0.0" rpy="1.57079 0.0 0.0"/>` 绕 `x` 轴旋转90度

```xml
<robot xmlns:xacro="http://www.ros.org/wiki/xacro">
     <xacro:macro name = "wheel_xacro" params = "wheel_name xyz">
        <link name="${wheel_name}_link">
            <visual>
                <!-- 沿着自己几何中心的偏移量和旋转量 -->
                <origin xyz="0.0 0.0 0.0" rpy="1.57079 0.0 0.0"/>
                <!-- 几何形状 -->
                <geometry>
                    <cylinder radius="0.032" length="0.04"/>
                </geometry>
                <!-- 材质颜色 -->
                <material name="yellow">
                    <color rgba="1.0 1.0 0.0 0.8"/>
                </material>
            </visual>
        </link>

        <joint name="${wheel_name}_joint" type="continuous">
            <parent link="base_link"/>
            <child link="${wheel_name}_link"/>
            <origin xyz="${xyz}" rpy="0.0 0.0 0.0"/>  
            <!-- 绕y轴旋转 -->
            <axis xyz="0 1 0"/>    
        </joint>
     </xacro:macro>
</robot>
```

在 `fishbot.urdf.xacro` 中创建轮子

```xml
<xacro:wheel_xacro wheel_name="left_wheel" xyz="0.0 0.1 -0.06"/>
<xacro:wheel_xacro wheel_name="right_wheel" xyz="0.0 -0.1 -0.06"/>
```

万向轮：

```xml
<xacro:macro name = "caster_xacro" params = "caster_name xyz">
        <link name="${caster_name}_link">
            <!-- 部件的外观描述 -->
            <visual>
                <!-- 沿着自己几何中心的偏移量和旋转量 -->
                <origin xyz="0.0 0.0 0.0" rpy="0.0 0.0 0.0"/>
                <!-- 几何形状 -->
                <geometry>
                    <sphere radius="0.016"/>
                </geometry>
                <!-- 材质颜色 -->
                <material name="yellow">
                    <color rgba="1.0 1.0 0.0 0.8"/>
                </material>
            </visual>
        </link>

        <joint name="${caster_name}_joint" type="fixed">
            <parent link="base_link"/>
            <child link="${caster_name}_link"/>
            <origin xyz="${xyz}" rpy="0.0 0.0 0.0"/>    
        </joint>
</xacro:macro>
```

在 `fishbot.urdf.xacro` 中创建万向轮

```xml
<xacro:caster_xacro caster_name="front_caster" xyz="0.08 0.0 -0.076"/>
<xacro:caster_xacro caster_name="back_caster" xyz="-0.08 0.0 -0.076"/>
```

#### 6.2.6 贴合地面，添加虚拟部件

因为之前的 `base_link` 位于中心，中心下面的部件嵌入地面，因此需要创建 `base_footprint` 部件位于地面，然后把 `base_link` 顶起来。 `- 0.001` 是为了把轮子向下嵌入1毫米

```xml
<!-- 虚拟部件 -->
<link name="base_footprint"/>
<joint name="joint_name" type="fixed">
    <origin xyz="0.0 0.0 ${length/2 + 0.032 - 0.001}" rpy="0.0 0.0 0.0"/>
    <parent link="base_footprint"/>
    <child link="base_link"/>
</joint>
```

