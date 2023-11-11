# Temp

可以在Flex中的sharedmemory脚本中,把构造函数打断点看一遍,到底哪里用了共享内存

| 验证                                                                                       | 结论                                                                                                         | 下一步打算                     |
|------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------|---------------------------|
| 数据驱动                                                                                     | 不管是eCAL还是SharedMemory发送那40个数据,都不能正常运行                                                                      | 去查看loadScene时候,Sofa相关代码   |
| eCAL在SerialPort中单独开个线程跑                                                                  | 会导致Flex延迟上升7-10ms,而且每次重置Vector会导致崩溃                                                                        | 放在main里再测试一下              |
| instrument_switch->m_pos_endoscope = &sofa_scene->m_posEndoscope; 在LoadScenes的时候,这里是NULL |                                                                                                            |                           |
| instrument_switch->m_tool_data需要看这里的调用,因为他接受来自serialPort的值                               |                                                                                                            |                           |
| 是InstrumentSwitch的Initialize先执行,还是SerialPort的先执行                                         | SerialPort的初始化先执行,创建了一个共享内存,可是在INstrument那里初始化的时候,却没有收到来自共享内存的值,这也是正常的,因为SerialPort是在Update里面执行接受来自共享内存的数据 | 所以要看看m_pos_endoscope的值的问题 |
| m_pos_endoscope需要验证一下他的执行流程                                                              | m_posEndoscope = TranslationMatrix(Point3(pos))*RotationMatrix(q);在sofascene里面会导致NAN                       | 跑sofa                     |
|                                                                                          |                                                                                                            |                           |
|                                                                                          |                                                                                                            |                           |
| 需要上机器跑一遍这个 (下班没人用机器后)                                                                    |                                                                                                            |                           |
|                                                                                          |                                                                                                            |                           |
| 在main里开线程跑eCAL的延迟                                                                        | 在std::cout40个值的情况下,依然26ms,增加了10ms,fuck!                                                                    | 看看怎么把延迟降低下来               |
| 在main里eCAL,清理vector会崩溃吗                                                                  |                                                                                                            |                           |
|                                                                                          |                                                                                                            |                           |
| 力反馈分离出来                                                                                  |                                                                                                            |                           |
|                                                                                          |                                                                                                            |                           |
| sofa中的力反馈,以及Geomagic中的力反馈,或者其他需要外部输入的参数,都找一下,做分离                                         |                                                                                                            |                           |
|                                                                                          |                                                                                                            |                           |
|                                                                                          |                                                                                                            |                           |
| 看geomagic和sofa中涉及力反馈数据接收的代码                                                              |                                                                                                            |                           |
|                                                                                          |                                                                                                            |                           |
| 把eCAL做成单独的场景                                                                             |                                                                                                            |                           |
| 把这个Flex工程做成相对路径的                                                                         |                                                                                                            |                           |

### HapticDeviceStatus.h

作为第一个加载的场景,直接创建了一个共享内存区域,可是这块共享内存,并没有人调用,也没copy to,也没copy from.

```C++
	HapticDeviceStatus(const char* name):Scene(name)
	{
		//创建共享内存，以场景的名字作为共享内存的名字
		m_sharedMemory = std::make_shared<SharedMemory>(mName, 10);
		TransmitData();
		std::cout << "[Haptic State]:" << m_hapticDeviceStatus << std::endl;
	}
```

同时这里有个值

```C++
int m_hapticDeviceStatus = 1;//力反馈状态，0：正常，1：异常
```

他的赋值是在`Calibration.h`
中,通过`geomagicCalibration->setHapticDeviceStatus(&hapticDeviceStatus->m_hapticDeviceStatus);`
完成的赋值.所归属的方法是`void Calibration(std::vector<shared_ptr<Scene>>& scenes, bool intergrated)`

**所以可以记录第一个力反馈方法,赋状态值这个**

| 力反馈方法                 |   |
|-----------------------|---|
| setHapticDeviceStatus |   |
|                       |   |
|                       |   |
|                       |   |

### GeomagicCalibration.h

在构造函数里,执行了`Init()`函数,这里执行了PhantomIoLib42.dll的服务循环

```C++
if (m_hdPhantomDriverDLL)
{
	//创建服务循环
	createServoLoop = (int(*)()) GetProcAddress(m_hdPhantomDriverDLL, "createServoLoop");
	//停止服务循环
	stopServoLoop = (int(*)())GetProcAddress(m_hdPhantomDriverDLL, "stopServoLoop");
	//销毁服务循环
	destroyServoLoop = (int(*)())GetProcAddress(m_hdPhantomDriverDLL, "destroyServoLoop");
	//初始化设备
	init_phantom = (int(*)(char *configname))GetProcAddress(m_hdPhantomDriverDLL, "init_phantom");
	//开始服务循环
	startServoLoop = (int(*)(int(_stdcall *fntServo)(void *), void *lpParam))GetProcAddress(m_hdPhantomDriverDLL, "startServoLoop");
	//取关节角度
	get_phantom_joint_angles = (int(*)(unsigned int index, float angles[6]))GetProcAddress(m_hdPhantomDriverDLL, "get_phantom_joint_angles");
	//取空间位置
	get_phantom_pos = (int(*)(unsigned int index, float pos[3]))GetProcAddress(m_hdPhantomDriverDLL, "get_phantom_pos");
	//更新数据
	update_phantom = (int(*)(unsigned int index))GetProcAddress(m_hdPhantomDriverDLL, "update_phantom");

	//校准
	update_calibration = (int(*)(unsigned int index))GetProcAddress(m_hdPhantomDriverDLL, "update_calibration");
	//关闭设备
	disable_phantom = (int(*)(unsigned int index))GetProcAddress(m_hdPhantomDriverDLL, "disable_phantom");

	m_nDevice = init_phantom("Default Device");
	cout << "力反馈设备句柄:" << m_nDevice << endl;
	if (m_nDevice < 0)
	{
		cout << "初始化力反馈失败" << endl;
		return;
	}

	cout << "初始化力反馈成功" << endl;
}
```

得到了一个`m_nDevice`这个是力反馈设备句柄,<0就是初始化失败,>0就是初始化成功,看看这里是不是要在没有的时候,默认作为成功
不对,这里应该分离出去,就是这里的`PhantomIoLib42.dll`放在虚拟Server里面,不要在这里直接进行校准,只把返回的结果传回来就可以了.
至于虚拟Server到底是传实际dll计算的值,还是直接发个成功,就Server说了算了.

同时,在`GeomagicCalibration`里面,同样调用了这个东西,需要改接口

```C++
static int _stdcall ServoRoutine(void *pParam)
{
	GeomagicCalibration *geomagicCalibration = (GeomagicCalibration*)pParam;
	geomagicCalibration->update_phantom(geomagicCalibration->m_nDevice);
	geomagicCalibration->get_phantom_joint_angles(geomagicCalibration->m_nDevice, geomagicCalibration->angles);
	geomagicCalibration->get_phantom_pos(geomagicCalibration->m_nDevice, geomagicCalibration->XYZ);
	return 0;
}
```

同样在`GeomagicCalibration`里面,更新校准结果的时候,也调用了这个

```C++
	void UpdateCalibration()
	{
		if (m_nDevice < 0)
		{
			cout << "力反馈异常，不进行校准" << endl;
			return;
		}

		update_calibration(m_nDevice);
		m_calibrationDone = true;
	}
```

有个疑问,

```C++
virtual void PostInitialize()
{
	g_sceneLower = Vec3(-5.0f, 0.0f, 0.0f);
	g_sceneUpper = g_sceneLower + Vec3(10.0f, 10.0f, 5.0f);
}
```

这里的这两个值,需要使用Server传递吗

然后有这么一个函数,设置力反馈设备状态的函数

```C++
	void setHapticDeviceStatus(int* hapticDeviceStatus)
	{
		m_hapticDeviceStatus = hapticDeviceStatus;
		if (m_hapticDeviceStatus)
		{
			if (m_nDevice >= 0)
				*m_hapticDeviceStatus = 0;//力反馈状态，0：正常，1：异常
		}
	}
```

这个方法只被一个地方调用,就是`Calibration.h`里面

```C++
	void Calibration(std::vector<shared_ptr<Scene>>& scenes, bool intergrated)
	{
		//������״̬��
		shared_ptr<HapticDeviceStatus> hapticDeviceStatus = std::make_shared<HapticDeviceStatus>("HapticDeviceStatus");
		hapticDeviceStatus->mIntergrated = intergrated;
		scenes.push_back(hapticDeviceStatus);

		shared_ptr<GeomagicCalibration> geomagicCalibration = std::make_shared<GeomagicCalibration>("GeomagicCalibration");
		geomagicCalibration->mIntergrated = intergrated;
		geomagicCalibration->m_useSharedMemory = true;
		scenes.push_back(geomagicCalibration);
		geomagicCalibration->setHapticDeviceStatus(&hapticDeviceStatus->m_hapticDeviceStatus);
	}
```

最后一行,调用了这个方法

然后这个脚本,即 `GeomagicCalibration.h`里面,的初始化函数`virtual void Initialize()`

```C++
if (m_nDevice >= 0)
{
	createServoLoop();
	startServoLoop(ServoRoutine, this);
}

//创建共享内存，以场景的名字作为共享内存的名字
if(m_useSharedMemory)
	m_sharedMemory = std::make_shared<SharedMemory>(mName, 20);
```

这里会创建服务循环,然后开启这个循环,并且新建了一个共享内存,并且在`Update`函数里面,接收了数据

```C++
std::vector<int> m_data;

m_sharedMemory->CopyFromSharedMemory(m_data, offset);
if (m_data.size())
{
	if (m_data[0] > m_count) // m_count==0
	{
		assert(m_data[0] = m_count + 1);
		cout << "校准次数：" << m_data[0] << endl;
		m_count = m_data[0];
		UpdateCalibration();
		cout << "校准完毕" << endl;
	}
```

这块共享内存不是从unity发的,谁发过来的啊,难道是给Unity发的??

其中`UpdateCalibration()`函数里面

```C++
update_calibration(m_nDevice);
```

会执行这个函数,这个函数也是来自于`PhantomIoLib42.dll`这个库.

这个脚本里面,比较重要的值是

```C++
float angles[9];
float XYZ[3];
```

这两个值的调用,这个是关节的角度,9个值.以及空间位置,3个数

```C++
geomagicCalibration->get_phantom_joint_angles(geomagicCalibration->m_nDevice, geomagicCalibration->angles);
geomagicCalibration->get_phantom_pos(geomagicCalibration->m_nDevice, geomagicCalibration->XYZ);
```

这两行代码,在初始化virtual void Initialize()函数里面调用

```C++
startServoLoop(ServoRoutine, this);
```

### Calibration.h

```C++
void Calibration(std::vector<shared_ptr<Scene>>& scenes, bool intergrated)
{
	//������״̬��
	shared_ptr<HapticDeviceStatus> hapticDeviceStatus = std::make_shared<HapticDeviceStatus>("HapticDeviceStatus");
	hapticDeviceStatus->mIntergrated = intergrated;
	scenes.push_back(hapticDeviceStatus);

	shared_ptr<GeomagicCalibration> geomagicCalibration = std::make_shared<GeomagicCalibration>("GeomagicCalibration");
	geomagicCalibration->mIntergrated = intergrated;
	geomagicCalibration->m_useSharedMemory = true;
	scenes.push_back(geomagicCalibration);
	geomagicCalibration->setHapticDeviceStatus(&hapticDeviceStatus->m_hapticDeviceStatus);
}
```

把这里分解下,里面的内容都填充好,每行干了什么,有什么是外部传进来的参数.

### Evaluation.h

```C++
virtual void Initialize()
{
	//创建共享内存，以场景的名字作为共享内存的名字
	m_sharedMemory = std::make_shared<SharedMemory>(mName, 1000);
}
```

也会创建一个共享内存

然后在Update函数里面调用了CopyTo

```C++
void Update()
{
	m_sharedMemory->CopyToSharedMemory(&evaluationData[0], evaluationData.size(), offset);
	m_sharedMemory->CopyToSharedMemory(&evaluationData[0], evaluationData.size(), offset);
	m_sharedMemory->CopyToSharedMemory(tipsPoints, offset);
}
```

在Update函数里面,把分解到电机的力通过共享内存发送给了Unity,以及抓取的软体类型和撕裂的软体类型,还有撕裂的比例,抓取的状态,血管类型.

```C++
	void Update()
	{
		vector<float> evaluationData;
		evaluationData.push_back(m_worldXZForce[0]);
		evaluationData.push_back(m_worldXZForce[1]);
		evaluationData.push_back(m_touchingSoftBodyType);
		evaluationData.push_back(m_tornSoftBodyType);

		evaluationData.push_back(m_tornProportion);
		evaluationData.push_back(m_graspingStatus);
		evaluationData.push_back(m_touchingVesselType);
		
		int offset = 0;
		m_sharedMemory->CopyToSharedMemory(&evaluationData[0], evaluationData.size(), offset);
		
		//出血点的信息也一起发过去
		m_sharedMemory->CopyToSharedMemory(bloodingPoints, offset);
		//还有提示点
		m_sharedMemory->CopyToSharedMemory(tipsPoints, offset);
```

其中的一些值

```C++
	Vec2 m_worldXZForce;//电机的力,方向是世界坐标的XZ轴
	int m_touchingSoftBodyType = 0;//没接触软体的话，默认为0，表示没有接触的软体
	int m_tornSoftBodyType = 0;//被撕裂的软体类型，默认为0，代表没有撕裂的软体
	float m_tornProportion = 0.0f;//软体撕裂百分比
	vector<pair<shared_ptr<Scene>, uint32_t>> m_bloodingPoints;
	vector<pair<shared_ptr<Scene>, uint32_t>> m_tipsPoints;

	int m_graspingStatus = 0;//抓取状态，0 ：没抓，1：抓取到软体没断，2：抓断软体，这功能其实跟被撕裂的软体类型重复
	int m_touchingVesselType = 0;//抓到的血管类型
```

### SerialPort

这个已经改造过了,不过考虑下,能不能完全用eCAL替换掉共享内存,或者同时先保留着,这样可以上真机,用SESS2.0去测试.

### Geomagic.h

这个脚本的构造函数里面,执行,通过程序校准定制的力反馈设备,但是由于`d_manualStart`给传的是false,所以不执行校准.

```C++
if (d_manualStart)
{
    //通过程序校准定制的力反馈设备
    m_geomagicCalibration = std::make_shared<GeomagicCalibration>("GeomagicCalibration");
    m_geomagicCalibration->Init();
    return;
}
initDevice();
```

只会执行`initDevice();`这个会初始化力反馈设备,应该用的是HD.dll,这个是不是自定义厂家,给自己做的dll啊

然后有个Update函数,每帧更新,大概是更新右边那个球体中心位置

```C++
//从设备的位置（m_posDevice）中提取旋转矩阵 rot 和平移向量 pos
Matrix33 rot(m_posDevice.GetAxis(0), m_posDevice.GetAxis(1), m_posDevice.GetAxis(2));
Vec3 pos = m_posDevice.GetTranslation();
```

然后有个KeyDown函数

```C++
case SDL_SCANCODE_KP_4:
    {
        //换器械
        if (m_instrumentSwitch.get())
            m_instrumentSwitch->m_instrument_name = "AblationCatheter";
        if (m_sofaScene.get())
            m_sofaScene->SwitchInstrument("InnerAblationCatheter.scn"); //键盘切换Sofa由于是异步操作，因此会有崩溃的情况发生
        break;
    }
```

其中做了换器械的操作,和Sofa有关.

```C++
 //输入数据
    std::string d_deviceName; ///<设备的名字

    Vec3 m_positionBase;
    Vec3 d_positionBase; ///<设备基座的位置

    Quat m_orientationBase;
    Quat d_orientationBase; ///<设备基座的姿态

    Quat d_orientationTool; ///<输入的器械的姿态
    double d_scale; ///<设备默认的坐标缩放比例
    double d_forceScale; ///<反馈力的缩放比例
    double d_maxInputForceFeedback; ///<允许最大的输入力，考虑安全性
    Vec3 d_inputForceFeedback;

    //输入参数
    bool d_manualStart;
    bool d_emitButtonEvent;
    bool d_frameVisu;
    bool d_omniVisu;
    bool d_useDHMethod; ///< 是否采用DH矩阵相乘的方法，如果否的话会直接采用API得出位姿

    Matrix44 m_posDevice;
    Matrix44 d_posDevice;

    Vec3 d_angle_1;
    Vec3 d_angle_2;

    Vec3 d_offsetBase; ///< 基座部分自由度角位移偏移
    Vec3 d_offsetBaseC; ///< 基座部分自由度角位移因子

    Vec3 d_offsetStylus; ///< 手柄部分自由度角位移偏移
    Vec3 d_offsetStylusC; ///< 手柄部分自由度角位移因子

    bool d_button_1;
    bool d_button_2;

    static bool s_schedulerRunning;
    static bool s_printCallback;
```

然后又以上输入数据,不知道是否可以通过eCAL来发送

还有个结构体

```C++
struct DeviceData
{
    HDdouble angle1[3];
    HDdouble angle2[3];
    HDdouble transform[16];
    int buttonState;
};
```

这里面的值,是不是也可以eCAL

```C++
bool m_simulationStarted;
    bool m_isInContact;
    DeviceData m_hapticData;
    DeviceData m_simuData;
    HHD m_hHD;
    std::vector<HDSchedulerHandle> m_hStateHandles;

    std::shared_ptr<InstrumentSwitch> m_instrumentSwitch;
    bool m_leftHandFrame = false;
    bool m_initDone = false;
    bool m_onlyZForce = false;
    Vec2* m_worldXZForce = nullptr;

    int m_posRotSetTimes = 0;
    int m_forceActivateTimes = 0;
    bool m_forceActivated = true;
```

这是在驱动和HD调度器之间成员变量交换

然后有几个静态的函数,不依赖于类

```C++
//获取第一个错误，如果没错，则返回HD_SUCCESS == 0
HDerror catchHDError(bool logError)
//回调函数，把力反馈设备的数据赋给仿真数据
HDCallbackCode HDCALLBACK copyDeviceDataCallback(void* pUserData)
//回调函数，获取笔尖的位置和姿态，并计算作用到笔尖的力
HDCallbackCode HDCALLBACK stateCallback(void* userData)
```

其中有个`driver->m_hapticData`
,这是要把力反馈数据赋值给仿真数据用,他的调用全部在最后一个静态函数里面`HDCallbackCode HDCALLBACK stateCallback(void* userData)`

**这函数里面,是有Sofa的耦合关系的.重点关注可以**

然后这个 `Geomagic.h`里面,还有两个函数,一个是    `void updatePosition()`,一个是 `void ComputePositions()`

第一个更新位置函数,也包含Sofa的耦合,他被`virtual void Initialize()`函数所调用一次.
在Tick()中,也被调用

然后Tick()会被Loop()调用, 然后Loop是在`virtual void Initialize()`里面,开了一个线程去运行的.

```C++
virtual void Initialize()
{
    updatePosition();
    //力反馈循环与Sofa同步，与Flex异步，否则会出现崩溃，并且影响力反馈手感
    m_loopThread = new std::thread(&Geomagic::Loop, this);
}
```

同时在Loop()里面,也存在和Sofa的耦合,

```C++
if (m_sofaScene)
{
    if (m_sofaScene->m_initialized)
    {
        m_sofaScene->BeforeTick();
        Tick();
        m_sofaScene->Tick();
    }
}
```

然后那个 `ComputePositions`函数,是在`UpdatePosition`里面调用的,以及下面这个函数里也调用了.

```C++
//回调函数，获取笔尖的位置和姿态，并计算作用到笔尖的力
HDCallbackCode HDCALLBACK stateCallback(void* userData)
```

总之,这个脚本充满了和Sofa的耦合关系,需要仔细的学习研究,然后做分离解耦.

### SofaScene.h

SofaScene的构造函数,啥也没干,空的.

然后从析构函数可以看出,开启过一个循环线程

有个函数 `void SetPosRotAdd(const Vec3& pAdd, const Quat& rAdd)`,作用是接收pos和qua然后转换为sofa的类型赋值给钳子

这个函数被`Geomagic.h`中的`Tick()`调用

```C++
if (int(toolData[16]) > m_posRotSetTimes)
{
  if (m_sofaScene.get())
   m_sofaScene->SetPosRotAdd(posAdd, rotAdd);
}
```
和支点相关


然后SofaScene同样有个初始化函数`virtual void Initialize()`,这里选择了具体对应的Main.scn文件,

获取场景的根节点 root，并从SOFA场景的0号网格中更新网格,获取Sofa场景的0号网格,创建gpu网格,从根节点中找到 SofaController::InnerInstrumentSwitch,
类型的对象 m_sofaInstrumentSwitch。同时写入新位置向量和旋转的四元数,处理器械偏移位姿（m_instrumentState0）、力反馈（m_forceFeedback）、内窥镜刚体组件（m_endoscopeState） 和套管刚体组件（m_tubeState）等组件，从根节点中找到对应的组件，如果找不到，将其打印出来
,搜索出Sofa场景中器械偏移前的实际位姿.

总之就是在这个初始化函数里面,完成了Sofa的初始化.


然后有个函数 `BeforeTick()`,首先，函数检查是否处于 "撕裂" (m_isTearing) 状态,如果是，那么先检查是否已添加了撕裂点 (m_addedTearPoint),
如果没有，那么会添加一个撕裂点，并将 m_addedTearPoint 设置为 true,//并且如果已经添加了撕裂点，那么清除撕裂点，并将 m_addedTearPoint 设置为 false,

总结一下，这个函数在每一 "tick"（时钟周期）之前被调用，以管理与 "撕裂" 相关的状态如果物体处于 "撕裂" 状态且尚未添加撕裂点，则添加一个撕裂点并计算撕裂力。如果物体不处于 "撕裂" 状态但已添加撕裂点，则清除撕裂点

然后这个函数,会被`Geomagic.h`中的`Loop`调用,以及在SofaScene.h中的Loop()函数中调用.

然后也有一个函数 `Tick()`,
```C++
	void Tick()
	{
		m_mainSimulation->step();
	}
```
这个函数同样,会被`Geomagic.h`中的`Loop`调用,以及在SofaScene.h中的Loop()函数中调用.

然后有个函数,`void setSpineCollisionGroup(bool active)`,函数的作用是激活或禁用名为 "Spine" 的子节点的冲突模型

这个函数会在`Geomagic.h`的`Tick()`函数中调用.和`int(toolData[25]`有关.

然后SofaScene同样有自己的Loop()函数.执行`BeforeTick();`和`Tick();`,只不过这个Loop并没有并任何函数调用.

然后有个函数 `void UpdateMeshFromSofaScene(Mesh* mesh, int index)`从主模拟环境 m_mainSimulation 中提取指定索引的网格数据并更新到提供的 mesh 对象
,从Sofa场景中获取索引为index的显示模型网格数据，赋给mesh,整个函数的主要目的是将主模拟环境中指定的网格数据更新到提供的 mesh 对象中

这个函数就是在`virtual void Initialize()`里面调用一次.

同样的,SofaScene有自己的Update函数,他会把器械,主镜,套管的位置更新出来,给别人使用.
//获取这个状态的姿态 pose。这个 pose 其中包含了旋转（orient）和中心（center）信息。
//把该信息转换成另一种格式的位姿数据，并通过一个变换矩阵更新物体的位置和旋转。

有个函数 `void SetPosture(float d)`
```C++
if (m_sofaInstrumentSwitch.get())
	m_sofaInstrumentSwitch->setPosture(d);
```
他只在`Geomagic.h`的`Tick()`函数中调用
```C++
m_sofaScene->SetPosture(toolData[15]);
```

然后还有个Move函数
```C++
void Move(float d)
{
	if (m_sofaInstrumentSwitch.get())
		m_sofaInstrumentSwitch->move(d);
}
```
也是会在`Geomagic.h`中调用,keyDown()里面调用
```C++
case SDL_SCANCODE_KP_0:
    {
        if (m_instrumentSwitch.get())
            m_instrumentSwitch->move(1.0);
        if (m_sofaScene.get())
            m_sofaScene->Move(1.0);
        break;
    }
```

有个函数 `void SwitchInstrument(string instrumentName)`,整个函数的作用是切换布局中的仪器，如果新的仪器名为空，或者根据名字无法找到对应的仪器，则会将当前的仪器状态设为 nullptr


这个函数的调用,依然全在`Geomagic.h`的`Tick()`里面,统一换器械那里会调用,以及KeyDown里面也调一遍,换器械用.

### InstrumentSwitch.h

构造啥也没干,然后有初始化函数 `void Initialize() override`

```C++
//创建主镜和套管，SofaScene和串口数据不存在的情况下，它们就不能创建了
//而且光有串口数据没有SofaScene的话，只能用于调试

if (m_pos_endoscope || m_tool_data)
{
    m_endoscope = make_shared<Endoscope>();
    m_endoscope->m_posDevice = m_pos_endoscope;
    m_endoscope->m_toolData = m_tool_data;
    m_endoscope->m_scale = m_scale;
    m_endoscope->Initialize();
}
if (m_pos_tube || m_tool_data){...}
if (m_send_data_)
{
    //创建共享内存
    m_shared_memory_ = std::make_shared<SharedMemory>(mName, 8);
}
```
创建了器械,同时开了一个共享内存,这个共享内存,只在 `void TransmitData() override`函数里面调了

```C++
if (m_send_data_)
{
    int offset = 0;
    vector<uint32_t> indices;
    indices.push_back(m_cur_instrument_index_);
    m_shared_memory_->CopyToSharedMemory(indices, offset);
}
```
同样具有Update函数
```C++
m_endoscope->GetToolData();
m_endoscope->Update();
m_tube->GetToolData();
m_tube->Update();

        //更新器械
        if (m_cur_instrument)
        {
            m_cur_instrument->GetToolData();
            m_cur_instrument->Update();
            collision_detect();
        }
```
对出血点,和切换器械,更新器械也有帮助,其中做了碰撞检测 `collision_detect();`

这个处理的是器械和软体的碰撞,并且计算出力来,**可以关注一下**




然后有个函数 `set_pos_rot_add(posAdd, rotAdd);`,在`Geomagic.h`的`Tick()`调用.

然后有个函数 `void TransmitData() override`
,会执行传递镜子,套管,当前器械(钳子)的数据,以及共享内存发送内存(这个不知道发给谁了,Unity?)

比如镜子的传递数据函数

```C++
    void TransmitData() override
    {
        int offset = 0;
        vector<Vec3> poses;
        poses.push_back(m_actualPos);
        m_sharedMemory->CopyToSharedMemory(poses, offset);

        vector<Quat> rots;
        rots.push_back(m_actualRot);
        m_sharedMemory->CopyToSharedMemory(rots, offset);
    }
```
这里去看看钉钉里的协议,是不是包含这一项,这就是器械的位置来回变换,来回传递,导致的问题吗,

然后有个加载器械的函数 `void load_instrument(const string &instrument_name)`

加载钳子,45度钳子,消融钳子,

这个只被下一个函数调用,`switch_instrument`切换钳子.这个只在update里面调用.

其中加载器械的时候,调用了一个`m_cur_instrument->Attach();`
比如消融钳子,
```C++
	virtual void Attach()
	{
		m_catheter->AddCollisionShape();
	}
	
	void AddCollisionShape()
	{
		if (!m_sdfFile.empty())
			AddSDF(m_sdfColliModel, m_pos + Rotate(m_rot, m_sdfRelativePos), m_rot, m_s);
	}
```


然后有个`void set_posture(const float d) const`函数,

会被`Geomagic.h`的Tick()调用.









### -------

```C++
instrumentSwitch->m_pos_endoscope = &sofaScene->m_posEndoscope;
instrumentSwitch->m_pos_tube = &sofaScene->m_posTube;
```

| Task                                            | Status |
|-------------------------------------------------|--------|
| 可以搞清楚SharedMemory的运行过程,然后我直接把SharedMemory给替换掉   |        |
| 需要搞清楚&sofaScene->m_posEndoscope他们两个的走向,是从哪里获取的值 |        |
| 一进去就启动eCAL,在Init或者其他地方就直接开启一个线程                 |        |
|                                                 |        |
|                                                 |        |

**In SerialPort Test**

```C++
const char *arguments[]{"1"};
char **argv = new char *[1];
argv[0] = new char[arguments[0]];

eCAL::Initialize(1, argv, "SerialPort subscriber");
eCAL::Process::SetState(proc_sev_healthy, proc_sev_level1, "I feel good");
eCAL::protobuf::CSubscriber<pb::SharedMemory::SerialPort> sub("SerialPort");

auto callback = std::bind(OnSerialPort, std::placeholders::_1, std::placeholders::_2, std::placeholders::_3,
                          std::placeholders::_4);
sub.AddReceiveCallback(callback);
while (eCAL::Ok())
{
}
eCAL::Finalize();

delete[] arguments;
delete[] argv;
```