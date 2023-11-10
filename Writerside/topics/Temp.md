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