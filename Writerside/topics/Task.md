# Task

|   | Verification item                                                                                                                          | Conclusion                                                                                                                                                                                                                                                                    | Next plan                                                            |
|---|--------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------|
| * | External data driven                                                                                                                       | Neither eCAL nor SharedMemory will drive the emulation properly.                                                                                                                                                                                                              | View loadScene and Sofa related codes                                |
| * | eCAL runs a separate thread in SerialPort                                                                                                  | This causes Flex to rise 7-10ms late, and each Vector reset causes a crash                                                                                                                                                                                                    | Change vector to array                                               |
|   | InstrumentSwitch and SerialPort, whose Initialize is executed first                                                                        | The initialization of SerialPort was performed first, creating a shared memory, but when initialized with the Instrument, it did not receive a value from the shared memory, which is normal because SerialPort is executed in Update and accepts data from the shared memory | So look at the value of m_pos_endoscope                              |
|   | m_pos_endoscope needs to verify its execution flow                                                                                         | m_posEndoscope = TranslationMatrix(Point3(pos))*RotationMatrix(q);Cause NaN in sofa scene                                                                                                                                                                                     | run sofa scene                                                       |
|   | Use a real machine to debug FleX                                                                                                           |                                                                                                                                                                                                                                                                               |                                                                      |
|   | The haptic needs to be isolated                                                                                                            |                                                                                                                                                                                                                                                                               |                                                                      |
|   | Force feedback in sofa, force feedback in Geomagic, or any other parameter that requires external input, look for it and do the separation |                                                                                                                                                                                                                                                                               |                                                                      |
|   | Look at the codes in geomagic and sofa that deal with the reception of force feedback data                                                 |                                                                                                                                                                                                                                                                               |                                                                      |
|   | Make eCAL a separate scene                                                                                                                 |                                                                                                                                                                                                                                                                               |                                                                      |
|   | Make the Flex project a relative path                                                                                                      |                                                                                                                                                                                                                                                                               |                                                                      |
|   | You can run through the shared memory case to see all the shared memory function calls, in what order, and by whom                         |                                                                                                                                                                                                                                                                               | LoadScenes does not read this array to array, it needs to be changed |
|   | Just change the part of SerialPort where eCAL accepts data,vector, to a fixed-length array, and then call the std::array                   | No additional performance cost                                                                                                                                                                                                                                                |                                                                      |
|   | Set the std::array address of tool_data in eCAL in SerialPort to LoadScene. Those two places need to be changed                            |                                                                                                                                                                                                                                                                               |                                                                      |
|   | What does the PostInitialize method do                                                                                                     |                                                                                                                                                                                                                                                                               |                                                                      |
|   | I can figure out how SharedMemory works, and then I can just replace SharedMemory                                                          |                                                                                                                                                                                                                                                                               |                                                                      |
|   | Encapsulate eCAL and use it as if calling shared memory.                                                                                   |                                                                                                                                                                                                                                                                               |                                                                      |
|   | eCAL has performance problem after using std::array<float,40>,need to check                                                                |                                                                                                                                                                                                                                                                               |                                                                      |
|   | Whether haptic needs to be separated from FleX                                                                                             |                                                                                                                                                                                                                                                                               |                                                                      |
|   | Analyze the driving logic of SOFA                                                                                                          |                                                                                                                                                                                                                                                                               |                                                                      |
|   | const std::array<float, 40> &toolData = *tool_data;   Check if &                                                                           |                                                                                                                                                                                                                                                                               |                                                                      |
 
## Learn

### A->a = &B->b;

```C++
geo_magic->m_worldXZForce = &evaluation->m_worldXZForce;
```

这行代码执行了一个赋值操作，将`evaluation`对象的成员变量`m_worldXZForce`的地址赋给了`geo_magic`
对象的成员变量`m_worldXZForce`。以下是解释：

- `geo_magic`是一个对象或指针，它有一个成员变量（或者可能是指针）`m_worldXZForce`。
- `evaluation`也是一个对象或指针，它有一个成员变量（或者可能是指针）`m_worldXZForce`。
- `&evaluation->m_worldXZForce`表示获取`evaluation`对象的`m_worldXZForce`成员的地址。

所以，这行代码的目的是将`evaluation`对象中`m_worldXZForce`的地址赋值给了`geo_magic`对象中的`m_worldXZForce`
。这样一来，两个对象的`m_worldXZForce`成员现在指向相同的内存位置，它们在程序中的改变将相互影响。

### GetProcAddress dll

```C++
// 这段 C++ 代码在Windows 的动态链接库（DLL）模块中查找一个名为 "createServoLoop" 的函数，并获取其地址
// m_hdPhantomDriverDLL 是一个包含 DLL 模块的句柄。这个句柄是通过 LoadLibrary 或者 GetModuleHandle 这样的函数获取的。
// GetProcAddress 是一个 Windows API 函数，它获取 DLL 模块中指定函数的地址.
// "createServoLoop" 是想要查找的函数名字.
// (int(*)()) 是一个函数指针类型转换。这表示该函数没有任何参数并返回一个整数。虽然 GetProcAddress 返回的是 FARPROC 类型，但通常我们会将其转换为正确的函数指针类型以便更安全地使用它。
// createServoLoop 是已经类型转换过的函数的地址存储变量。
// 总的来说，这段代码的功能是获取动态链接库中名为 "createServoLoop" 的函数的地址，并将其存储在 createServoLoop 变量中。通过这种方式，您可以在运行时调用 DLL 中的函数，而不需要在编译时知道这些函数的存在.
createServoLoop = (int(*)()) GetProcAddress(m_hdPhantomDriverDLL, "createServoLoop");//创建服务循环
```

### _stdcall

```C++
	//该函数似乎被设计成用作多线程应用程序的线程入口点或回调函数
	//该函数声明为静态，并且预期遵循_stdcall调用约定
	//总体而言，这个函数似乎是一个更大程序或系统的一部分，涉及与某个设备（可能是Geomagic触觉设备）交互，
	//以更新其状态并检索有关其关节角度和位置的信息。鉴于其结构和static关键字的使用，该函数可能被设计为在单独的线程中运行
	//它采用_stdcall调用约定，这是一种在Windows平台上常见的函数调用约定。
	static int _stdcall ServoRoutine(void *pParam)
	{
		GeomagicCalibration *geomagicCalibration = (GeomagicCalibration*)pParam;//将pParam指针强制转换为GeomagicCalibration类型的指针。这表明pParam预计指向GeomagicCalibration类或结构的实例
		geomagicCalibration->update_phantom(geomagicCalibration->m_nDevice);
		geomagicCalibration->get_phantom_joint_angles(geomagicCalibration->m_nDevice, geomagicCalibration->angles);
		geomagicCalibration->get_phantom_pos(geomagicCalibration->m_nDevice, geomagicCalibration->XYZ);
		return 0;
	}
```

### std::array<float, 40> &toolData = *tool_data;
这行代码是在C++中创建一个引用到`std::array`的实例的语句。

`std::array`是C++的一种容器，用于保存固定数量的元素。在这个例子中，`std::array<float, 40>`是一个包含40个`float`类型元素的数组。

`tool_data`是一个指向`std::array<float, 40>`类型的指针。

`*tool_data`是对指针`tool_data`进行解引用操作，获取指针指向的实际对象（即本例中的`std::array<float, 40>`类型的对象）。

`std::array<float, 40> &toolData = *tool_data;`这行代码的含义是，创建一个名为`toolData`的引用，引用的对象是`tool_data`指针指向的`std::array<float, 40>`对象。在之后的代码中，你可以直接使用`toolData`来操作`tool_data`指向的数组，而不需要每次都进行解引用操作。


### std::array<float, 40> tool_data_array = *tool_data;

这行C++代码做的是从一个指向`std::array<float, 40>`类型的指针`tool_data`中取出数组，并将其复制到`tool_data_array`变量中。

`std::array<float, 40>`定义了一个包含40个`float`类型元素的数组。

`*tool_data`对指针`tool_data`进行解引用操作，获取指针指向的实际对象（即`std::array<float, 40>`类型的对象）。

`std::array<float, 40> tool_data_array = *tool_data;`这行代码的含义是，创建一个新的`std::array<float, 40>`类型的变量`tool_data_array`，并将`tool_data`指针指向的`std::array<float, 40>`对象复制到`tool_data_array`中。这是深复制，也就是说，`tool_data_array`中的元素和`tool_data`指向的数组中的元素值相同，但是它们是两份独立的数据，修改其中一个并不会影响另一个。

