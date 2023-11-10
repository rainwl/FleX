# eCAL Test

## Proto
**sharedmemory.proto**
```C++
syntax = "proto3";

import "coord.proto";
import "misc.proto";

package pb.SharedMemory;

message SerialPort
{
    Coordinate.Vector3 EndoscopePos = 1;
    Coordinate.Euler EndoscopeEuler = 2;
    Coordinate.Vector3 TubePos = 3;
    Coordinate.Euler TubeEuler = 4;
    Misc.Offset offset = 5;
    Coordinate.Quaternion RotCoord = 6;
    Coordinate.Vector3 PivotPos = 7;
    int32 AblationCount = 8;
    Misc.Haptic Haptic = 9;
    int32 HemostasisCount = 10;
    int32 HemostasisIndex = 11;
    Misc.SoftTissue SoftTissue = 12;
    int32 NerveRootDance = 13;
}
```
**coord.proto**

```C++
syntax = "proto3";

package pb.Coordinate;

message Vector3
{
    double x = 1;
    double y = 2;
    double z = 3;
}

message Euler
{
    double x = 1;
    double y = 2;
    double z = 3;
}

message Quaternion
{
    double x = 1;
    double y = 2;
    double z = 3;
    double w = 4;
}
```
**misc.proto**
```C++
syntax = "proto3";

package pb.Misc;

message SoftTissue
{
    int32 LigamentumFlavum = 1;
    int32 DiscYellowSpace = 2;
    int32 VeutroVessel = 3;
    int32 Fat = 4;
    int32 FibrousRings = 5;
    int32 NucleusPulposus = 6;
    int32 PosteriorLongitudinalLigament = 7;
    int32 DuraMater = 8;
    int32 NerveRoot = 9;
}

message Haptic
{
    int32 HapticState = 1;
    double HapticOffset = 2;
    double HapticForce = 3;
}

message Offset
{
    double EndoscopeOffset = 1;
    double TubeOffset = 2;
    int32 InstrumentSwitch = 3;
    int32 AnimationValue = 4;
    double PivotOffset = 5;
}
```

## Snd

```C++
#include <ecal/ecal.h>
#include <ecal/msg/protobuf/publisher.h>
#include <iostream>
#include "sharedmemory.pb.h"
#include <random>

int main(int argc, char **argv)
{
    eCAL::Initialize(argc, argv, "SerialPort publisher");
    eCAL::Process::SetState(proc_sev_healthy, proc_sev_level1, "I feel good !");
    eCAL::protobuf::CPublisher<pb::SharedMemory::SerialPort> pub("SerialPort");
    pb::SharedMemory::SerialPort serial;

    std::random_device rd;
    std::mt19937 gen(rd());
    std::uniform_real_distribution<double> dis(0.0,180.0);
    std::uniform_int_distribution<int> dis_int(1,20);   

    auto cnt = 0;
    while (eCAL::Ok())
    {
        serial.mutable_endoscopepos()->set_x(dis(gen));
        serial.mutable_endoscopepos()->set_y(dis(gen));
        serial.mutable_endoscopepos()->set_z(dis(gen));

        serial.mutable_endoscopeeuler()->set_x(dis(gen));
        serial.mutable_endoscopeeuler()->set_y(dis(gen));
        serial.mutable_endoscopeeuler()->set_z(dis(gen));

        serial.mutable_tubepos()->set_x(dis(gen));
        serial.mutable_tubepos()->set_y(dis(gen));
        serial.mutable_tubepos()->set_z(dis(gen));

        serial.mutable_tubeeuler()->set_x(dis(gen));
        serial.mutable_tubeeuler()->set_y(dis(gen));
        serial.mutable_tubeeuler()->set_z(dis(gen));

        serial.mutable_offset()->set_endoscopeoffset(dis(gen));
        serial.mutable_offset()->set_tubeoffset(dis(gen));
        serial.mutable_offset()->set_instrumentswitch(dis_int(gen));
        serial.mutable_offset()->set_animationvalue(dis_int(gen));
        serial.mutable_offset()->set_pivotoffset(dis(gen));

        serial.mutable_rotcoord()->set_x(dis(gen));
        serial.mutable_rotcoord()->set_y(dis(gen));
        serial.mutable_rotcoord()->set_z(dis(gen));
        serial.mutable_rotcoord()->set_w(dis(gen));

        serial.mutable_pivotpos()->set_x(dis(gen));
        serial.mutable_pivotpos()->set_y(dis(gen));
        serial.mutable_pivotpos()->set_z(dis(gen));

        serial.set_ablationcount(0);

        serial.mutable_haptic()->set_hapticstate(dis_int(gen));
        serial.mutable_haptic()->set_hapticoffset(dis(gen));
        serial.mutable_haptic()->set_hapticforce(dis(gen));

        serial.set_hemostasiscount(dis_int(gen));
        serial.set_hemostasisindex(dis_int(gen));

        serial.mutable_softtissue()->set_ligamentumflavum(0);
        serial.mutable_softtissue()->set_discyellowspace(0);
        serial.mutable_softtissue()->set_veutrovessel(0);
        serial.mutable_softtissue()->set_fat(0);
        serial.mutable_softtissue()->set_fibrousrings(0);
        serial.mutable_softtissue()->set_nucleuspulposus(0);
        serial.mutable_softtissue()->set_posteriorlongitudinalligament(0);
        serial.mutable_softtissue()->set_duramater(0);
        serial.mutable_softtissue()->set_nerveroot(0);

        serial.set_nerverootdance(1);

        pub.Send(serial);

        // print content
        std::cout << "EndoscopePos     : " << serial.endoscopepos().x() << "," << serial.endoscopepos().y() << "," <<serial.endoscopepos().z() << std::endl;
        std::cout << "EndoscopeEuler   : " << serial.endoscopeeuler().x() << "," << serial.endoscopeeuler().y() << ","<< serial.endoscopeeuler().z() << std::endl;
        std::cout << "TubePos          : " << serial.tubepos().x() << "," << serial.tubepos().y() << "," << serial.tubepos().z() <<std::endl;
        std::cout << "TubeEuler        : " << serial.tubeeuler().x() << "," << serial.tubeeuler().y() << "," << serial.tubeeuler().z()<< std::endl;
        std::cout << "EndoscopeOffset  : " << serial.offset().endoscopeoffset() << std::endl;
        std::cout << "TubeOffset       : " << serial.offset().tubeoffset() << std::endl;
        std::cout << "InstrumentSwitch : " << serial.offset().instrumentswitch() << std::endl;
        std::cout << "AnimationValue   : " << serial.offset().animationvalue() << std::endl;
        std::cout << "PivotOffset      : " << serial.offset().pivotoffset() << std::endl;
        std::cout << "RotCoordinate    : " << serial.rotcoord().x() << "," << serial.rotcoord().y() << "," << serial.rotcoord().z() << "," << serial.rotcoord().w() << std::endl;
        std::cout << "PivotPos         : " << serial.pivotpos().x() << "," << serial.pivotpos().y() << "," << serial.pivotpos().z() << std::endl;
        std::cout << "AblationCount    : " << serial.ablationcount() << std::endl;
        std::cout << "HapticState      : " << serial.haptic().hapticstate() << std::endl;
        std::cout << "HapticOffset     : " << serial.haptic().hapticoffset() << std::endl;
        std::cout << "HapticForce      : " << serial.haptic().hapticforce() << std::endl;
        std::cout << "HemostasisCount  : " << serial.hemostasiscount() << std::endl;
        std::cout << "HemostasisIndex  : " << serial.hemostasisindex() << std::endl;
        std::cout << "LigamentumFlavum : " << serial.softtissue().ligamentumflavum() << std::endl;
        std::cout << "DiscYellowSpace  : " << serial.softtissue().discyellowspace() << std::endl;
        std::cout << "VeutroVessel     : " << serial.softtissue().veutrovessel() << std::endl;
        std::cout << "Fat              : " << serial.softtissue().fat() << std::endl;
        std::cout << "FibrousRings     : " << serial.softtissue().fibrousrings() << std::endl;
        std::cout << "NucleusPulposus  : " << serial.softtissue().nucleuspulposus() << std::endl;
        std::cout << "Posterior        : " << serial.softtissue().posteriorlongitudinalligament() << std::endl;
        std::cout << "DuraMater        : " << serial.softtissue().duramater() << std::endl;
        std::cout << "NerveRoot        : " << serial.softtissue().nerveroot() << std::endl;
        std::cout << "NerveRootDance   : " << serial.nerverootdance() << std::endl;

        std::cout << std::endl;

        // sleep 500 ms
        eCAL::Process::SleepMS(500);
    }

    eCAL::Finalize();

    return (0);
}

```

## Rec

```C++
#include <ecal/ecal.h>
#include <ecal/msg/protobuf/subscriber.h>
#include <iostream>
#include "sharedmemory.pb.h"

void OnSerialPort(const char* topic_name_, const pb::SharedMemory::SerialPort& serial, const long long time_, const long long clock_)
{
	std::cout << "------------------------------------------" << std::endl;
	std::cout << " HEAD " << std::endl;
	std::cout << "------------------------------------------" << std::endl;
	std::cout << "topic name   : " << topic_name_ << std::endl;
	std::cout << "topic time   : " << time_ << std::endl;
	std::cout << "topic clock  : " << clock_ << std::endl;
	std::cout << "------------------------------------------" << std::endl;
	std::cout << " CONTENT " << std::endl;
	std::cout << "------------------------------------------" << std::endl;
    std::cout << "EndoscopePos     : " << serial.endoscopepos().x() << "," << serial.endoscopepos().y() << "," <<serial.endoscopepos().z() << std::endl;
    std::cout << "EndoscopeEuler   : " << serial.endoscopeeuler().x() << "," << serial.endoscopeeuler().y() << ","<< serial.endoscopeeuler().z() << std::endl;
    std::cout << "TubePos          : " << serial.tubepos().x() << "," << serial.tubepos().y() << "," << serial.tubepos().z() <<std::endl;
    std::cout << "TubeEuler        : " << serial.tubeeuler().x() << "," << serial.tubeeuler().y() << "," << serial.tubeeuler().z()<< std::endl;
    std::cout << "EndoscopeOffset  : " << serial.offset().endoscopeoffset() << std::endl;
    std::cout << "TubeOffset       : " << serial.offset().tubeoffset() << std::endl;
    std::cout << "InstrumentSwitch : " << serial.offset().instrumentswitch() << std::endl;
    std::cout << "AnimationValue   : " << serial.offset().animationvalue() << std::endl;
    std::cout << "PivotOffset      : " << serial.offset().pivotoffset() << std::endl;
    std::cout << "RotCoordinate    : " << serial.rotcoord().x() << "," << serial.rotcoord().y() << "," << serial.rotcoord().z() << "," << serial.rotcoord().w() << std::endl;
    std::cout << "PivotPos         : " << serial.pivotpos().x() << "," << serial.pivotpos().y() << "," << serial.pivotpos().z() << std::endl;
    std::cout << "AblationCount    : " << serial.ablationcount() << std::endl;
    std::cout << "HapticState      : " << serial.haptic().hapticstate() << std::endl;
    std::cout << "HapticOffset     : " << serial.haptic().hapticoffset() << std::endl;
    std::cout << "HapticForce      : " << serial.haptic().hapticforce() << std::endl;
    std::cout << "HemostasisCount  : " << serial.hemostasiscount() << std::endl;
    std::cout << "HemostasisIndex  : " << serial.hemostasisindex() << std::endl;
    std::cout << "LigamentumFlavum : " << serial.softtissue().ligamentumflavum() << std::endl;
    std::cout << "DiscYellowSpace  : " << serial.softtissue().discyellowspace() << std::endl;
    std::cout << "VeutroVessel     : " << serial.softtissue().veutrovessel() << std::endl;
    std::cout << "Fat              : " << serial.softtissue().fat() << std::endl;
    std::cout << "FibrousRings     : " << serial.softtissue().fibrousrings() << std::endl;
    std::cout << "NucleusPulposus  : " << serial.softtissue().nucleuspulposus() << std::endl;
    std::cout << "Posterior        : " << serial.softtissue().posteriorlongitudinalligament() << std::endl;
    std::cout << "DuraMater        : " << serial.softtissue().duramater() << std::endl;
    std::cout << "NerveRoot        : " << serial.softtissue().nerveroot() << std::endl;
    std::cout << "NerveRootDance   : " << serial.nerverootdance() << std::endl;
	std::cout << "------------------------------------------" << std::endl;
	std::cout << std::endl;
}

int main(int argc, char** argv)
{
	eCAL::Initialize(argc, argv, "SerialPort subscriber");
	eCAL::Process::SetState(proc_sev_healthy, proc_sev_level1, "I feel good !");
	eCAL::protobuf::CSubscriber<pb::SharedMemory::SerialPort> sub("SerialPort");

	auto callback = std::bind(OnSerialPort, std::placeholders::_1, std::placeholders::_2, std::placeholders::_3, std::placeholders::_4);
	sub.AddReceiveCallback(callback);

	while (eCAL::Ok())
	{
		// sleep 100 ms
		eCAL::Process::SleepMS(100);
	}

	eCAL::Finalize();

	return(0);
}

```