# eCAL Reference

This paper introduces two construction methods for eCAL+protobuf:

Methods
: MSBuild
: CMake

Finally, the test code is given.

## Pure sln

```Bash
└───Snd
    ├───include
    │   ├───ecal
    │   └───google
    ├───lib
    │   ├───ecal_cored.lib
    │   ├───ecal_protod.lib
    │   └───libprotobufd.lib
    └───protobuf
        ├───*.pb.cc
        └───*.pb.h
```

- `ecal` is copied from `C:\eCAL\include\ecal`
- `google` is copied from `C:\eCAL\include\google`
- `*.lib` is copied from `C:\eCAL\lib`

> Do not copy the `google` directory directly from the `protobuf repository`.
> Because their versions don't agree.
> {style="note"}

### Configuration Properties

#### C/C++->General->Additional Include Directories

```Bash
.\protobuf
.\include
```

#### Linker->Input->Additional Dependencies

```Bash
.\lib\ecal_cored.lib
.\lib\libprotobufd.lib
.\lib\ecal_protod.lib
kernel32.lib
user32.lib
gdi32.lib
winspool.lib
shell32.lib
ole32.lib
oleaut32.lib
uuid.lib
comdlg32.lib
advapi32.lib
```
#### Add Exiting item...

Don't forget to add these items under `/protobuf` to the project:

```Bash
hello_world.pb.cc
hello_world.pb.h
```


#### Test Code:Snd

```C++
#include <ecal/ecal.h>
#include <ecal/msg/protobuf/publisher.h>

#include <iostream>
#include <thread>

#include "protobuf/hello_world.pb.h"

int main(int argc, char** argv)
{
    // Initialize eCAL and create a protobuf publisher
    eCAL::Initialize(argc, argv, "Hello World Protobuf Publisher");
    eCAL::protobuf::CPublisher<proto_messages::HelloWorld> publisher("hello_world_protobuf");

    // Ask the user to input his name
    std::cout << "Please enter your name: ";
    std::string name;
    std::getline(std::cin, name);

    unsigned int id = 0;

    // Infinite loop (using eCAL::Ok() will enable us to gracefully shutdown the
    // Process from another application)
    while (eCAL::Ok())
    {
        // Let the user input a message
        std::cout << "Type the message you want to send: ";
        std::string message;
        std::getline(std::cin, message);

        // Create a protobuf message object
        proto_messages::HelloWorld hello_world_message;
        hello_world_message.set_name(name);
        hello_world_message.set_msg(message);
        hello_world_message.set_id(id++);

        // Send the message
        publisher.Send(hello_world_message);
        std::cout << "Sent message!" << std::endl << std::endl;
    }

    // finalize eCAL API
    eCAL::Finalize();
}

```

#### Test Code:Rec

```C++
#include <ecal/ecal.h>
#include <ecal/msg/protobuf/subscriber.h>

#include <iostream>
#include <thread>

#include "hello_world.pb.h"

void HelloWorldCallback(const proto_messages::HelloWorld& hello_world_msg)
{
    std::cout << hello_world_msg.name() << " sent a message with ID "
        << hello_world_msg.id() << ":" << std::endl
        << hello_world_msg.msg() << std::endl << std::endl;
}

int main(int argc, char** argv)
{
    // Initialize eCAL and create a protobuf subscriber
    eCAL::Initialize(argc, argv, "Hello World Protobuf Subscriber");
    eCAL::protobuf::CSubscriber<proto_messages::HelloWorld> subscriber("hello_world_protobuf");

    // Set the Callback
    subscriber.AddReceiveCallback(std::bind(&HelloWorldCallback, std::placeholders::_2));

    // Just don't exit
    while (eCAL::Ok())
    {
        std::this_thread::sleep_for(std::chrono::milliseconds(500));
    }

    // finalize eCAL API
    eCAL::Finalize();
}
```

## CMake + sln

**Directory structure**
```
├─  proto_messages
│  └─  hello_world.proto
│
├─  protobuf_snd
│  ├─  CMakeLists.txt
│  └─  main.cpp
│
└─  protobuf_rec
   ├─  CMakeLists.txt
   └─  main.cpp
```

`proto_messages/hello_world.proto`
```C++
syntax = "proto3";

package proto_messages;

message HelloWorld
{
  string name      = 1;
  uint32 id        = 2;
  string msg       = 3;
}
```
### Publisher

`CMakeLists.txt`

```CMake
cmake_minimum_required(VERSION 3.0)
set(CMAKE_FIND_PACKAGE_PREFER_CONFIG ON)

project(protobuf_snd)

set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

find_package(eCAL REQUIRED)
find_package(Protobuf REQUIRED)

set(source_files
  main.cpp
)

set(protobuf_files
    ${CMAKE_CURRENT_SOURCE_DIR}/../proto_messages/hello_world.proto
)

add_executable(${PROJECT_NAME} ${source_files})

PROTOBUF_TARGET_CPP(${PROJECT_NAME} ${CMAKE_CURRENT_SOURCE_DIR}/../proto_messages/ ${protobuf_files})

target_link_libraries(${PROJECT_NAME}
  eCAL::core
  protobuf::libprotobuf
)
```

`main.cpp`

```C++
#include <ecal/ecal.h>
#include <ecal/msg/protobuf/publisher.h>

#include <iostream>
#include <thread>

#include "hello_world.pb.h"

int main(int argc, char** argv)
{
  // Initialize eCAL and create a protobuf publisher
  eCAL::Initialize(argc, argv, "Hello World Protobuf Publisher");
  eCAL::protobuf::CPublisher<proto_messages::HelloWorld> publisher("hello_world_protobuf");

  // Ask the user to input his name
  std::cout << "Please enter your name: ";
  std::string name;
  std::getline(std::cin, name);

  unsigned int id = 0;

  // Infinite loop (using eCAL::Ok() will enable us to gracefully shutdown the
  // Process from another application)
  while (eCAL::Ok())
  {
    // Let the user input a message
    std::cout << "Type the message you want to send: ";
    std::string message;
    std::getline(std::cin, message);

    // Create a protobuf message object
    proto_messages::HelloWorld hello_world_message;
    hello_world_message.set_name(name);
    hello_world_message.set_msg (message);
    hello_world_message.set_id  (id++);

    // Send the message
    publisher.Send(hello_world_message);
    std::cout << "Sent message!" << std::endl << std::endl;
  }

  // finalize eCAL API
  eCAL::Finalize();
}
```
### Subscriber

`CMakeLists.txt`

```CMake
cmake_minimum_required(VERSION 3.0)
set(CMAKE_FIND_PACKAGE_PREFER_CONFIG ON)

project(protobuf_rec)

set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

find_package(eCAL REQUIRED)
find_package(Protobuf REQUIRED)

set(source_files
  main.cpp
)

set(protobuf_files
    ${CMAKE_CURRENT_SOURCE_DIR}/../proto_messages/hello_world.proto
)

add_executable(${PROJECT_NAME} ${source_files})

PROTOBUF_TARGET_CPP(${PROJECT_NAME} ${CMAKE_CURRENT_SOURCE_DIR}/../proto_messages/ ${protobuf_files})

target_link_libraries(${PROJECT_NAME}
  eCAL::core
  protobuf::libprotobuf
)
```

`main.cpp`

```C++
#include <ecal/ecal.h>
#include <ecal/msg/protobuf/subscriber.h>

#include <iostream>
#include <thread>

#include "hello_world.pb.h"

void HelloWorldCallback(const proto_messages::HelloWorld& hello_world_msg)
{
  std::cout << hello_world_msg.name() << " sent a message with ID "
            << hello_world_msg.id() << ":" << std::endl
            << hello_world_msg.msg() << std::endl << std::endl;
}

int main(int argc, char** argv)
{
  // Initialize eCAL and create a protobuf subscriber
  eCAL::Initialize(argc, argv, "Hello World Protobuf Subscriber");
  eCAL::protobuf::CSubscriber<proto_messages::HelloWorld> subscriber("hello_world_protobuf");

  // Set the Callback
  subscriber.AddReceiveCallback(std::bind(&HelloWorldCallback, std::placeholders::_2));

  // Just don't exit
  while (eCAL::Ok())
  {
    std::this_thread::sleep_for(std::chrono::milliseconds(500));
  }

  // finalize eCAL API
  eCAL::Finalize();
}
```

## Generate pb.cc and pb.h

Because we ues `eCAL`,so the `protoc.exe`,we just use what `eCAL` provides.

In `Edit the system environment varialbes`,open the `Environment Variables`,
choose the `Path` under the `User variables for {user}`,click `Edit`,and add a
new one:
```Bash
C:\eCAL\bin
```
Once this is set up, we can use the following command to generate pb files at any time.

```Bash
protoc --cpp_out=. myproto.proto
```

`or`

use this for build all *.proto files.

```Bash
protoc --cpp_out=. *.proto
```

## Testing

### protobuf

![](eCAL_protobuf.png) {width="500"}

### Rec
```C++
#include <ecal/ecal.h>
#include <ecal/msg/protobuf/subscriber.h>
#include <iostream>
#include "person.pb.h"

void OnPerson(const char* topic_name_, const pb::People::Person& person_, const long long time_, const long long clock_)
{
  std::cout << "------------------------------------------" << std::endl;
  std::cout << " HEAD "                                     << std::endl;
  std::cout << "------------------------------------------" << std::endl;
  std::cout << "topic name   : " << topic_name_             << std::endl;
  std::cout << "topic time   : " << time_                   << std::endl;
  std::cout << "topic clock  : " << clock_                  << std::endl;
  std::cout << "------------------------------------------" << std::endl;
  std::cout << " CONTENT "                                  << std::endl;
  std::cout << "------------------------------------------" << std::endl;
  std::cout << "person id    : " << person_.id()            << std::endl;
  std::cout << "person name  : " << person_.name()          << std::endl;
  std::cout << "person stype : " << person_.stype()         << std::endl;
  std::cout << "person email : " << person_.email()         << std::endl;
  std::cout << "dog.name     : " << person_.dog().name()    << std::endl;
  std::cout << "house.rooms  : " << person_.house().rooms() << std::endl;
  std::cout << "------------------------------------------" << std::endl;
  std::cout                                                 << std::endl;
}

int main(int argc, char **argv)
{
  // initialize eCAL API
  eCAL::Initialize(argc, argv, "person subscriber");

  // set process state
  eCAL::Process::SetState(proc_sev_healthy, proc_sev_level1, "I feel good !");

  // create a subscriber (topic name "person")
  eCAL::protobuf::CSubscriber<pb::People::Person> sub("person");

  // add receive callback function (_1 = topic_name, _2 = msg, _3 = time, _4 = clock, _5 = id)
  auto callback = std::bind(OnPerson, std::placeholders::_1, std::placeholders::_2, std::placeholders::_3, std::placeholders::_4);
  sub.AddReceiveCallback(callback);

  while(eCAL::Ok())
  {
    // sleep 100 ms
    eCAL::Process::SleepMS(100);
  }

  // finalize eCAL API
  eCAL::Finalize();
  
  return(0);
}

```

### Snd

```C++
#include <ecal/ecal.h>
#include <ecal/msg/protobuf/publisher.h>
#include <iostream>
#include "person.pb.h"

int main(int argc, char **argv)
{
  eCAL::Initialize(argc, argv, "person publisher");// initialize eCAL API
  eCAL::Process::SetState(proc_sev_healthy, proc_sev_level1, "I feel good !");// set process state
}

```
**single or dynamic publisher**
```C++
  // create a publisher (topic name "person")
  eCAL::protobuf::CPublisher<pb::People::Person> pub("person");
  // generate a class instance of Person
  pb::People::Person person;
```
`or`
```C++
  // create a dynamic publisher (topic name "person")
  eCAL::protobuf::CDynamicPublisher pub1("person", std::make_shared<pb::People::Person>());
  std::shared_ptr<pb::People::Person> person1 = pub1.GetAs<pb::People::Person>();

  // create a dynamic publisher (topic name "person")
  eCAL::protobuf::CDynamicPublisher pub2("person", "pb.People.Person");
  std::shared_ptr<pb::People::Person> person2 = pub2.GetAs<pb::People::Person>();
  
  // set person1 object content
  person1->set_name("Max");
  person1->set_stype(pb::People::Person_SType_MALE);
  person1->set_email("max@mail.net");
  person1->mutable_dog()->set_name("Brandy");
  person1->mutable_house()->set_rooms(4);

  // set person2 object content
  person2->set_name("Romy");
  person2->set_stype(pb::People::Person_SType_FEMALE);
  person2->set_email("romy@mail.net");
  person2->mutable_dog()->set_name("Gorky");
  person2->mutable_house()->set_rooms(4);
```

**enter main loop**
```C++
  auto cnt = 0;
  while(eCAL::Ok())
  {
    // set person object content
    person.set_id(++cnt);
    person.set_name("Max");
    person.set_stype(pb::People::Person_SType_MALE);
    person.set_email("max@mail.net");
    person.mutable_dog()->set_name("Brandy");
    person.mutable_house()->set_rooms(4);

    // send the person object
    pub.Send(person);

    // print content
    std::cout << "person id    : " << person.id()            << std::endl;
    std::cout << "person name  : " << person.name()          << std::endl;
    std::cout << "person stype : " << person.stype()         << std::endl;
    std::cout << "person email : " << person.email()         << std::endl;
    std::cout << "dog.name     : " << person.dog().name()    << std::endl;
    std::cout << "house.rooms  : " << person.house().rooms() << std::endl;
    std::cout                                                << std::endl;

    // sleep 500 ms
    eCAL::Process::SleepMS(500);
  }

  // finalize eCAL API
  eCAL::Finalize();

  return(0);
```
`or`
```C++
  auto cnt = 0;
  std::shared_ptr<pb::People::Person> person;

  while(eCAL::Ok())
  {
    if (++cnt % 2)
    {
      // modify and send the person1 object
      person1->set_id(cnt);
      pub1.Send();

      // for later printing
      person = person1;
    }
    else
    {
      // modify and send the person2 object
      person2->set_id(cnt);
      pub2.Send();

      // for later printing
      person = person2;
    }

    // print current person message
    std::cout << "person id    : " << person->id()            << std::endl;
    std::cout << "person name  : " << person->name()          << std::endl;
    std::cout << "person stype : " << person->stype()         << std::endl;
    std::cout << "person email : " << person->email()         << std::endl;
    std::cout << "dog.name     : " << person->dog().name()    << std::endl;
    std::cout << "house.rooms  : " << person->house().rooms() << std::endl;
    std::cout << std::endl;

    // sleep 500 ms
    eCAL::Process::SleepMS(500);
  }

  // finalize eCAL API
  eCAL::Finalize();

  return(0);
```