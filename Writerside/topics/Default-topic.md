# Simulation Server


## Overview

This article focuses on the development of the `Simulation Server` and
its adaptation to the FleX `business requirements`.

## Simulation Server

Network Transfer
: Using a network transfer allows we to send data between different devices without a physical connection.

Data Format
: Using Protocol Buffer (provisional)

Latency and Performance
: UDP

Testing and Debugging
: Log (consider this issue later)

**NetWork**

| Library | use | reason                  |
|---------|-----|-------------------------|
| POCO    | x   | do not support protobuf |
| libhv   | x   | not good enough         |
| evpp    | x   | not good enough         |
| ecal    | √   |                         |

**Protocol**

|                 |   |                                    |
|-----------------|---|------------------------------------|
| Cap’n Proto     | x | https://capnproto.org/             |
| Protocol Buffer | √ | https://github.com/protocolbuffers |

> `Cap'n Proto`:After hours of reading the documentation and building the source code, as well as watching instructional
> videos on YouTube, I found that the API, performance aside, was poorly documented, and the examples were not good
> enough
> to understand.
> {style="warning"}

> eCAL + Protocol Buffer is all right
>
> https://github.com/eclipse-ecal/ecal
>
> https://eclipse-ecal.github.io/ecal/getting_started/hello_world.html#
> {style="note"}

**Render**

|       |   |                                                                                        |
|-------|---|----------------------------------------------------------------------------------------|
| IMGUI |   | https://github.com/ocornut/imgui                                                       |
|       |   | https://www.bilibili.com/video/BV13e4y1m73N/?spm_id_from=333.337.search-card.all.click |
| GODOT |   |                                                                                        |

## Business Requirements

### General Interface

![](https://pic4rain.oss-cn-beijing.aliyuncs.com/img/tool_data_trans.png)

in `SerialPort.h`,we create a new variable named `tool_data`

```C++
public:
    void Update() override
    {
        int offset = 0;
        shared_memory_->CopyFromSharedMemory(tool_data_, offset);
        tool_data = tool_data_;
    }
    std::vector<float> tool_data;
    
private:
    std::vector<float> tool_data_;
    std::shared_ptr<SharedMemory> shared_memory_;
```

> naming style refer to Google C++ Style
> {style="note"}

In `LoadScenes.h`

```C++
void LoadScenes(std::vector<shared_ptr<Scene>>& scenes, bool intergrated)
{
    geomagic->m_toolData = &serialPort->tool_data;
    instrumentSwitch->m_tool_data = &serialPort->tool_data;
}
```

### Protocol

```C++
auto callback = std::bind(&SerialPort::on_serial_port, this, std::placeholders::_1, std::placeholders::_2,std::placeholders::_3,std::placeholders::_4);
```

### SerialPort (Unity->Flex)

| value    | description  | code                                | issue   |
|----------|--------------|-------------------------------------|---------|
| 2.000015 | 主镜位置0        | Endoscope.h::m_actualPos            | dont in |
| 3.000009 | 主镜位置1        | Endoscope.h::m_actualPos            | dont in |
| 7.999690 | 主镜位置2        | Endoscope.h::m_actualPos            | dont in |
| 338.5304 | 主镜欧拉角3       | Geomagic.h::rx                      | dont in |
| 13.23317 | 主镜欧拉角4       | Geomagic.h::ry                      | dont in |
| 11.95613 | 主镜欧拉角5       | Geomagic.h::zEndoscopeRot           | dont in |
| 1.000009 | 套管位置6        |                                     |         |
| 2.000009 | 套管位置7        |                                     |         |
| 5.999765 | 套管位置8        |                                     |         |
| 338.5304 | 套管欧拉角9       |                                     |         |
| 13.23317 | 套管欧拉角10      |                                     |         |
| 17.15376 | 套管欧拉角11      | Geomagic.h::zTubeRot                | dont in |
| -1.22898 | 主镜上下偏移量12    | Geomagic.h::zEndoscope              | dont in |
| -3.61975 | 套管上下偏移量13    | Geomagic.h::zTube                   | dont in |
| 60,      | 换头编号14       | Geomagic.h::instrumentIndex         | ok      |
| 0.9,     | 钳子动画值15      | Geomagic.h::m_instrumentSwitch      | ok      |
| 2,       | 支点变量值16      |                                     |         |
| 0,       | 四元数17        |                                     |         |
| 0.707106 | 四元数18        |                                     |         |
| 0,       | 四元数19        |                                     |         |
| 0.707106 | 四元数20        |                                     |         |
| -10,     | 支点数值21       |                                     |         |
| 4.9,     | 支点数值22       |                                     |         |
| -0.9,    | 支点数值23       |                                     |         |
| 0,       | 消融次数 24      |                                     |         |
| 3,       | 力反馈是否给力      |                                     |         |
| -1,      | 力反馈钳子偏移量26   | Geomagic::deltaZ                    | dont in |
| 2,       | 力反馈力的大小27    |                                     |         |
| 0,       | 止血次数(累加次数)28 |                                     |         |
| 0,       | 止血点索引29      | InstrumentSwitch.h::point_index     | ok      |
| 1,       | 黄韧带 30       |                                     |         |
| 1,       | 盘黄间隙31       |                                     |         |
| 1,       | 腹侧血管32       |                                     |         |
| 1,       | 脂肪33         |                                     |         |
| 1,       | 纤维环34        |                                     |         |
| 1,       | 髓核35         |                                     |         |
| 1,       | 后纵韧带36       |                                     |         |
| 1,       | 硬膜囊37        |                                     |         |
| 1,       | 神经根38        |                                     |         |
| 0        | 神经根跳动39      | InstrumentSwitch.h::m_nerve_beating | ok      |

```Bash
2.000015,  //主镜位置0
3.000009,  //主镜位置1
7.999690,  //主镜位置2
338.5304,  //主镜欧拉角3
13.23317,  //主镜欧拉角4
11.95613,  //主镜欧拉角5
1.000009,  //套管位置6
2.000009,  //套管位置7
5.999765,  //套管位置8
338.5304,  //套管欧拉角9
13.23317,  //套管欧拉角10
17.15376,  //套管欧拉角11
-1.228983, //主镜上下偏移量12
-3.619757, //套管上下偏移量13
60,        //换头编号14
0.9,       //钳子动画值15
2,         //支点变量值16
0,         //四元数17
0.7071068, //四元数18
0,         //四元数19
0.7071068, //四元数20
-10,       //支点数值21
4.9,       //支点数值22
-0.9,      //支点数值23
0,         //消融次数 24
3,         //力反馈是否给力 0:关闭力反馈,1:开启力反馈,2:力反馈物理全关闭,3:力反馈物理全开启
-1,        //力反馈钳子偏移量26
2,         //力反馈力的大小27
0,         //止血次数(累加次数)28
0,         //止血点索引(止的是哪一个出血点)29
1,         //黄韧带 撕扯软体的状态：0~1  (0:不能抓,1:能抓)30
1,         //盘黄间隙31
1,         //腹侧血管32
1,         //脂肪33
1,         //纤维环34
1,         //髓核35
1,         //后纵韧带36
1,         //硬膜囊37
1,         //神经根38
0          //神经根跳动39
```

