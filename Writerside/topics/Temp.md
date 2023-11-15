# Haptic&Sofa

## LoadScenes

Here is the scene loading order and configuration.

| Scene              |
|--------------------|
| HapticDeviceStatus |
| Evaluation         |
| SerialPort         |
| Geomagic           |
| SofaScene          |
| InstrumentSwitch   |

| object            | point                       | Value                                       |
|-------------------|-----------------------------|---------------------------------------------|
| geo_magic         | ->m_toolData                | &serial_port->tool_data                     |
| geo_magic         | ->m_worldXZForce            | &evaluation->m_worldXZForce                 |
| geo_magic         | ->setHapticDeviceStatus( )  | &haptic_device_status->m_hapticDeviceStatus |
| geo_magic         | ->m_sofaScene               | sofa_scene                                  |
| ---               | ---                         | ---                                         |
| instrument_switch | ->m_pos_add                 | sofa_scene->m_posAdd                        |
| instrument_switch | ->m_rot_add                 | sofa_scene->m_rotAdd                        |
| instrument_switch | ->m_pos_device              | &sofa_scene->m_posDevice                    |
| instrument_switch | ->m_is_tearing              | &sofa_scene->m_isTearing                    |
| instrument_switch | ->m_tear_force              | &sofa_scene->m_tearForce                    |
| instrument_switch | ->m_pos_endoscope           | &sofa_scene->m_posEndoscope                 |
| instrument_switch | ->m_pos_tube                | &sofa_scene->m_posTube                      |
| instrument_switch | ->m_tool_data               | &serial_port->tool_data                     |
| instrument_switch | ->m_touching_soft_body_type | &evaluation->m_touchingSoftBodyType         |
| instrument_switch | ->m_torn_soft_body_type     | &evaluation->m_tornSoftBodyType             |
| instrument_switch | ->m_torn_proportion         | &evaluation->m_tornProportion               |
| instrument_switch | ->m_grasping_status         | &evaluation->m_graspingStatus               |
| instrument_switch | ->m_touching_vessel_type    | &evaluation->m_touchingVesselType           |
| instrument_switch | ->m_blooding_points         | &evaluation->m_bloodingPoints               |
| instrument_switch | ->m_tips_points             | &evaluation->m_tipsPoints                   |
| ---               | ---                         | ---                                         |
| geo_magic         | ->m_instrumentSwitch        | instrument_switch                           |

## Decoupling

Here are some values to consider for the emulated server.

| Value                      | Type     | Class                 | Description               |
|----------------------------|----------|-----------------------|---------------------------|
| m_hapticDeviceStatus       | int      | HapticDeviceStatus.h  | 0:normal 1:abnormal       |
| Init()                     | function | GeomagicCalibration.h | PhantomIoLib42.dll        |
| m_nDevice                  | int      | GeomagicCalibration.h |                           |
| GeomagicCalibration.h      | class    | GeomagicCalibration.h | External call             |
| Geomagic.h                 | class    | Geomagic.h            | External call             |
| void updatePosition()      | function | Geomagic.h            | Strong coupling with SOFA |
| void Tick()                | function | Geomagic.h            | Strong coupling with SOFA |
| void ComputePositions()    | function | Geomagic.h            | Strong coupling with SOFA |
| void Initialize() override |          | SofaScene.h           |                           |
| void BeforeTick()          |          | SofaScene.h           | Tear and compute force    |
|                            |          |                       |                           |
|                            |          |                       |                           |

## HapticDeviceStatus.h

![](HapticDeviceStatus_refactor.png)

After combing through the original code structure, I reconstructed the header file.

The variable `m_hapticDeviceStatus` has been called in the following code.

```C++
geomagic_calibration->
setHapticDeviceStatus(&haptic_device_status->m_hapticDeviceStatus);

geo_magic->
setHapticDeviceStatus(&haptic_device_status->m_hapticDeviceStatus);
```

## GeomagicCalibration.h

![](GeomagicCalibration.png)

This class is only called when doing Haptic calibration.So we can consider extracting the entire class and calling it on
the simulated server.

```C++
//Calibration.h
namespace spline_surgery{
void Calibration(std::vector<shared_ptr<Scene>>& scenes, bool intergrated)
{
	const auto geomagic_calibration = 
	std::make_shared<GeomagicCalibration>("GeomagicCalibration");
	geomagic_calibration->mIntergrated = intergrated;
	geomagic_calibration->m_useSharedMemory = true;
	scenes.push_back(geomagic_calibration);
	geomagic_calibration->
	set_haptic_device_status(&haptic_device_status->m_hapticDeviceStatus);
}}
```

## Calibration.h

![](Calibration.png)

The calibration part should be completely separate.

## Evaluation.h

![](Evaluation.png)

There is nothing to analyze in this class. Pay attention to the motor force.

## SerialPort

![](SerialPort.png)

## Geomagic.h

![](Geomagic.png)

This class and SOFA coupling is extremely strong, focus on.

## SofaScene.h

![](SofaScene.png)

## InstrumentSwitch.h

![](InstrumentSwitch.png)

