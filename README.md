# drone_control

## Communication

**simulation**

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b1e4f6d0-6859-412d-aad7-1c7621b6d97e/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b1e4f6d0-6859-412d-aad7-1c7621b6d97e/Untitled.png)

**Companion Computer & Pixhawk4**

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/550a30ec-ed97-4ab6-92dc-80da19bc617f/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/550a30ec-ed97-4ab6-92dc-80da19bc617f/Untitled.png)

## **Pixhawk4 Specification**

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0295d583-ec97-4664-8761-bf4eb16582ac/pixhawk.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0295d583-ec97-4664-8761-bf4eb16582ac/pixhawk.png)

[Spec](https://www.notion.so/83de6e37d65f445f9ce2d24f9e9ffac4)

## **Introduction**

Pixhawk는 일종의 고성능 비행 제어 컴퓨터로, 고정익 비행기, 멀티콥터, 헬리 콥터와 더불어 로봇, 자동차, 보트 등 다양한 플랫폼에 적용이 가능하다. 또한, 하드웨어와 소프트웨어까지 모두 오픈되어 다양한 연구자들과 기업들이 개발을 할 수 있는 환경이 구성되었다. Pixhawk는 처음 개발 시 드론 시스템을 개발하였기 때문에 단순히 비행 조종 컴퓨터만 있는 것이 아니라 GCS(Ground Control System), Log Viewer, HILS(Hardware In The Loop), SITL(Software In The Loop), MAVLink와 같은 지상국 시스템과 시뮬레이션 시스템 등이 제공된다.

간단히 살펴보면, GCS는 지상국 시스템으로 비행 상태 확인, 지도와의 연동이 가능하고 Qt 기반으로 개발되어 Window, Linux, Mac, Android 등에서도 활용이 가능하다. Log Viewer는 비행 시험 분석에 이용되고 Java로 개발된 FlightPlot이라는 프로그램이 있다. HILS는 Pixhawk만 연결하면 드론이 없어도 가상으로 드론을 날려볼 수 있는 시스템으로, 드론을 동작하게 하는 모든 프로세서는 Pixhawk 내부에서 이루어지고 센서 정보와 시각 정보만 HILS 시스템에서 할 수 있게 된다. STIL는 Pixhawk도 필요하지 않고 컴퓨터 내에서 모든 시뮬레이션이 가능하며, Gazebo와 연동하여 현실적인 시스템을 구현할 수 있다. 보통 실제 비행 전에 SITL시뮬레이션을 먼저 진행하고 HITL 시뮬레이션을 진행한다. MAVLink는 드론의 표준화된 통신 프로토콜이라고 생각할 수 있는데, 헤더 오버헤드가 8byte로 매우 가벼운 편이고 프로토콜 추가도 쉽다. 또한 ROS와의 연동을 가능하게 한다.

## **Develop Environment**

Ubuntu 18.04

ROS Melodic

Gazebo 9

## **Pixhawk Source Code Build**

우선, Pixhawk에 대한 패키지들을 설치한다.

```
>> sudo apt-get install python-argparse git-core wget zip  python-empy qtcreator cmake build-essential genromfs -y
```

다음으로는 시뮬레이션에 대한 패키지들을 설치한다.

```
>> sudo apt-get install ant protobuf-compiler libeigen3-dev libopencv-dev openjdk-8-jdk openjdk-8-jre clang lldb -y
```

Pixhawk를 실제로 사용한다면 추가적인 작업이 필요하다. 

우선, 시리얼 포트 연결 권한을 주기 위해 user mode로 변경을 하여 매번 펌웨어를 업로드 할 때마다 관리자 권한이 필요하지 않도록 한다.

```
sudo usermod -a -G dialout $USER  
```

다음으로는 우분투에서 관리하는 시리얼 포트 관리를 제거한다.

```
sudo apt-get remove modemmanager  
```

다음으로는 ARM 계열의 Pixhawk에 맞춰 컴파일을 할 수 있도록 크로스 컴파일 환경을 만들어준다.

```
sudo add-apt-repository ppa:terry.guo/gcc-arm-embedded -y  sudo apt-get update  sudo apt-get install python-serial openocd \      flex bison libncurses5-dev autoconf texinfo build-essential \    libftdi-dev libtool zlib1g-dev \    python-empy gcc-arm-none-eabi -y
```

이제 Pixhawk에서 제공하는 소스코드를 다운받는다. 

(추가적으로 stable version인 1.2.0을 다운받으려면 두번째 줄과 같이 branch 명령어를 사용한다.) 

(하지만, 현재 1.2.0 버전은 비공개로 되어있는 것 같다. Master 버전으로 해도 무방하다.)

(Pixhawk 소스코드는 최상위 디렉토리에 설치하는 것이 좋은 것 같다. 괜히 workspace 안에 설치하면 build 시에 에러를 발생하는 경우가 종종 발생한다. 또한 Pixhawk 소스코드는 빌드할 필요가 없기 때문에 workspace와 따로 두는 것이 안정적이라고 생각한다.)

```
git clone https://github.com/PX4/Firmware.git  git clone https://github.com/PX4/Firmware.git --branch v1.2.0  
```

다음으로 Pixhawk 하드웨어로 진행하는 경우는 첫번째 줄, 하드웨어 없이 시뮬레이션 기반으로 진행하는 경우는 두번째 줄과 같이 빌드한다. 

(Pixhawk4의 경우에는 px4_fmu-v5_default를 사용하라고 한다.)

- **[Pixhawk	4](https://docs.px4.io/master/en/flight_controller/pixhawk4.html)**:	`make	px4_fmu-v5_default`
- **[Pixhawk	4 Mini](https://docs.px4.io/master/en/flight_controller/pixhawk4_mini.html)**:	`make	px4_fmu-v5_default`
- **[CUAV	V5+](https://docs.px4.io/master/en/flight_controller/cuav_v5_plus.html)**:	`make	px4_fmu-v5_default`
- **[CUAV	V5 nano](https://docs.px4.io/master/en/flight_controller/cuav_v5_nano.html)**:	`make	px4_fmu-v5_default`
- **[Pixracer](https://docs.px4.io/master/en/flight_controller/pixracer.html)**:	`make	px4_fmu-v4_default`
- **[Pixhawk	3 Pro](https://docs.px4.io/master/en/flight_controller/pixhawk3_pro.html)**:	`make	px4_fmu-v4pro_default`
- **[Pixhawk	Mini](https://docs.px4.io/master/en/flight_controller/pixhawk_mini.html)**:	`make	px4_fmu-v3_default`
- **[Pixhawk	2 (Cube Black)](https://docs.px4.io/master/en/flight_controller/pixhawk-2.html)**:	`make	px4_fmu-v3_default`
- **[mRo	Pixhawk](https://docs.px4.io/master/en/flight_controller/mro_pixhawk.html)**:	`make	px4_fmu-v3_default` (supports	2MB Flash)
- **[Holybro	pix32](https://docs.px4.io/master/en/flight_controller/holybro_pix32.html)**:	`make	px4_fmu-v2_default`
- **[Pixfalcon](https://docs.px4.io/master/en/flight_controller/pixfalcon.html)**:	`make	px4_fmu-v2_default`
- **[Dropix](https://docs.px4.io/master/en/flight_controller/dropix.html)**:	`make	px4_fmu-v2_default`

(버전이 올라가면서 posix라는 명령어는 사라지고 px4_sitl_default를 사용하라고 한다.)
(recipe for target 'gazebo' failed 에러가 발생하는 경우에 다음의 패키지를 설치한다.
sudo apt install libgstreamer1.0-dev)
```
>>> make px4fmu-v2_default>>> make posix_sitl_default
```

## **Dive into Pixhawk Source Code**

오픈 소스를 활용하기 위해서 전체 구조를 파악하는 것이 중요하다. 또한, README 파일은 많은 정보가 함축되어 있을 가능성이 높기 때문에 자세히 살펴본다.

**Debug**

Pixhawk는 JTAG를 이용하여 디버깅할 수 있고 OpenOCD와 GDB를 사용한다. 하지만, 이미 Bootloader가 있고, Firmware는 USB를 통해 시리얼로 업로드가 가능하므로 알고리즘 개발이 위주라면 굳이 건드리지 않아도 된다.

**Documentation**

Pixhawk는 내부 코드가 Doxygen 방법으로 주석이 달려있다. Doxygen은 주석을 분석하여 PDF나 html 문서로 자동 생성해 준다. 최근엔 업데이트가 잘 안된다.

**Images**

Pixhawk를 컴파일해서 펌웨어 이미지를 만들기 위해 필요한 정보들이 있다. 여러 종류의 Pixhawk 하드우에어에 맞춰 필요한 정보들이 .prototype이라는 파일로 들어있다. 이 정보를 사용하여 이미지를 만들면 .bin이라는 형태로 생성된다.

**NuttX**

Pixhawk가 사용하는 실시간 운영체제이고 이를 기반으로 Pixhawk가 동작된다. 별도의 다른 오픈소스이므로 가져다 쓰면 된다. 하드웨어 속성 변경이 필요할 경우 NuttX 설정을 변경할 필요가 생기는데 nuttx-config 디렉토리에서 변경 가능하다.

**ROMFS**

ROM FileSystem을 의미하며 Pixhawk 펌웨어가올라가면 파일 시스템이 생기는데 여기에 파일들이
존재하게 된다. 중요한 부분 중 하나이고, 이곳에 Pixhawk가 처음 동작되는 것을 알려주는 rcS 스크립트가
들어있다. 그 외에도, 각 기체마다 필요한 설정값들, 파라미터 등이 포함된다.

**Tools**

스크립트 형태로 Source 컴파일 시 필요한 스크립트, 코딩 스타일 체크 등의 스크립트가 들어있다. 또한, 앞으로  SITL을 위한 Gazebo에 필요한 코드들도 여기에 포함되어 있다. 이 코드 또한 별도 오픈 소스로 개발되고 있어서 NuttX과 같은 방식으로 보면 된다. 

(조금은 건드릴 수도 있다.)

**Cmake**

Pixhawk 내부 컴파일 과정이 포함되어 있다. 여러 모듈 중 선택적으로 컴파일을 할 수 있도록 설정이 가능하고 새로운 모듈을 생성해 추가하는 것도 이곳에서 한다.

**Integrationtests**

컴파일 후 기본적인 기능에 대해 테스트하는 부분이다. 

(중요하진 않다.)

**Launch**

SITL을 ROS와 연동하기 위해 개발된 부분이다. 최근 ROS없이 SITL 동작이 가능하지만, 본인의 경우에는 ROS를 사용해 제어해야 하므로 필요하다.

**MAVLink**

지상국 시스템과 Pixhawk 기반 드론의 통신 프로토콜로 MAVLink라는 오픈 소스 코드를 그래도 사용한다. 만약 새로운 통신 프로토콜을 만들려면 이 부분을 수정해야 한다.

**Misc**

분류되지 않은 부분들이 들어있다고 보면 된다. 현재는 tones라는 소리를 정의하고 있다. 이는 기체 상태를 간단한 소리를 통해 알려주는 방법인데 Buzzer를 생각하면 된다.

**Msg**

uORB의 메세지를 정의한다. uORB란, 모듈과 모듈간의 통신을 하기 위한 기반 시스템이고, 모듈간 통신 프로토콜의 정의가 msg 디렉토리 안에 들어있다. 텍스트 타입의 메세지를 header파일로 변경되는데 이는 Tools의 스크립트가 담당한다.

**Nuttx-configs**

NuttX에 대한 설정들이 포함되어 있다. Pixhawk는 기본적으로 Serial Baud Rate가 57600인데이 부분은 운영체제에서 담당하기 때문에 이를 바꾸기 위해서 nuttx-config를 수정해야 한다.

**Posix-configs**

우리가 사용할 SITL을 위해 동작하게 될 Ubuntu 운영체제가 할 일들이 나와있다. 실제적으로 동작시킬 프로그램들 내용이 포함된다.

**Src**

수정해야 할 소스 코드가 들어있는 디렉토리이다. 워낙 양이 많기 때문에 코드 중 필요한 모듈을 찾아 분석해 수정하는 것이 좋다.

- Drivers: 주로	센서 드라이버가 있다. 추가적으로	센서를 드론에 추가하려면 이 부분에 모듈을 추가하면	된다.
- Example: 예제	샘플 코드가 존재한다.
- Firmware: 설치에	필요한 설정이 있다. 중요하진	않다.
- Include: 일부	헤더 파일들이 들어있고, 기본이	되는 px4.h가	이곳에 존재한다.
- Lib: task로	수행되진 않고 task에	사용될 라이브러리들이 포함되어 있다.
- Modules: 위치	예측, 자세제어	등 핵심 모듈들이 들어있는 디렉토리이다.
- Platforms: nuttx와	posix를	구분하여 각 운영체제에 맞춰 필요한 라이브러리들이	여기에 포함되어 있다. 필요한	이유는 각 플랫폼마다 같은 기능인데 함수 이름이	다른 경우가 있기 때문이다.
- Systemcmds: 시스템에	필요한 명령어에 대한 소스 코드가 여기 포함되어	있다. Version 정보를	보는 ver 명령어나	parameter 정보를	보는 param 명령어가	그 중 하나이다.

**Unittests**

말 그대로 Unit test를 하기 위한 코드들이 존재한다.




## **MAVROS**

SITL에서 roslaunch를 통해 실행되는 노드는 mavros와 pub_setpoints_position인데,
처음 roscore를 실행시키고 두 노드를 실행시키면 roscore를 통해 publish와 subscribe을 하게 된다. pub_setpoints_position에 있는 명령들을 publish하면 mavros가 subscribe하고 그 내용을 PX4에 전달한다. 또한, mavros 노드가 PX4의 topic들을 subscribe하기 때문에 센서, 속도, 위치 등을 모니터링 할 수 있게 된다. 

MAVROS/MAVLink 설치 방법은 공식 홈페이지를 참고한다.

(가장 먼저 MAVROS/MAVLink가 포함된 workspace를 만드는 것이 안정적이다.) 

(또한 catkin_make와 catkin build는 섞어서 사용이 불가능한 것 같다. Catkin build가 모듈을 개별적으로  uild가 가능하기 때문에 catkin build를 사용하는 것이 더 유용하다고 생각한다.)

[https://docs.px4.io/master/en/ros/mavros_installation.html](https://docs.px4.io/master/en/ros/mavros_installation.html)

그 후에 github에서 MAVROS test를 위해 clone한다.

```
$ cd ~/catkin_ws/src$ git clone https://github.com/Jaeyoung-Lim/modudculab_ros.git$ catkin build modudculab_ros$ source home/nile/catkin_ws/devel/setup.bash
```

다음으로 위와 같이 빌드를 진행하면 되는데, 빌드가 잘 되지 않을 것이다.
(modudculab_ros/CmakeLists.txt 파일의 mavros dependency를 추가해야 하기 때문이다.)

**gedit modudculab_ros/CMakeLists.txt # add mavros_msgs**

find_package(catkin REQUIRED COMPONENTS

roscpp

rospy

std_msgs

mavros_msgs

)

**gedit modudculab_ros/package.xml # add mavros_msgs**

<buildtool_depend>catkin</buildtool_depend>

<build_depend>roscpp</build_depend>

<build_depend>rospy</build_depend>

<build_depend>std_msgs</build_depend>

<build_depend>mavros_msgs</build_depend>

<run_depend>roscpp</run_depend>

<run_depend>rospy</run_depend>

<run_depend>std_msgs</run_depend>

<run_depend>mavros_msgs</run_depend>

만약, 빌드를 마쳤다면 make px4_sitl_default gazebo를 실행시키고 roslaunch파일을 실행시킨다.

(하지만 정상적으로 실행되지 않는데 이는 launch파일을 수정해줘야 한다.)

**gedit modudculab_ros/launch/ctrl_pos_gazebo.launch**

<arg name="fcu_url" default= "udp://:14540@127.0.0.1:14557">

<!-- <arg name="fcu_url" default= "/dev/ttyUSB0:921600" /> -->

```
$ roscore$ roslaunch modudculab_ros ctrl_pos_gazebo.launch$ rosrun mavros mavsafety arm$ rosrun mavros mavsys mode -c OFFBOARD
```

이를 마치면, 드론의 호버링을 확인할 수 있다.

## **MAVLink**

SITL 파트에서 보았듯이 Pixhawk, Gazebo, QGroundControl은 MAVLink 메세지를 통해 UDP통신을 한다. TCP/IP는 상대방과 페어링이 되어야 통신이 되는 반면, UDP는 페어링이 필요없기 때문에 데이터의 유실 가능성이 존재한다.

posix-configs/SITL/init/rcSgazeboiris 파일을 보면 동작 태스크를 알 수 있다.

```
mavlink start -u 14556 -r 2000000  mavlink start -u 14557 -r 2000000 -m onboard -o 14540  mavlink stream -r 80 -s POSITION_TARGET_LOCAL_NED -u 14556  mavlink stream -r 80 -s LOCAL_POSITION_NED -u 14556  
```

MAVLink 명령어는 Pixhawk에서 MAVLink 메세지를 수신하고 송신하는 기능을 한다. 데몬 형식으로 계속 수행되는 방식인데, 이 데몬 형식을 수행시키는 것이 “mavlink start”이다. “-u 15446”은 UDP로 통신을 하고 15466번 포트로 MAVLink 데이터를 수신받겠다는 의미이다. 그리고 “-o 14540”은 14540 포트로 MAVLink 데이터를 송신하겠다는 의미이다. “-o” 옵션이 없는 경우는 기본적으로 14540 포트를 쓰겠다는 의미이고, 위에서 mavlink start가 두개가 실행된 것은 수신해야 할 서로 다른 데이터가 2개 있다는 뜻으로 해석할 수 있다. 하나는 지상국 시스템이고 다른 하나는 “-m onboard”를 통해 보았을 때, Companion Board라고 추측해 볼 수 있을 것이다. “mavlink stream”은 외부로부터 MAVLink 스트림 데이터를 수신하겠다는 것으로 이해하면 된다.

## MAVROS wiki

[http://wiki.ros.org/mavros#Subscribed_Topics](http://wiki.ros.org/mavros#Subscribed_Topics)

arm, takeoff, land같은 미션 명령은 service로 전달되고, IMU, GPS 데이터의 경우에는 topic으로 전달된다. (코드를 보며 더 살펴봐야 할 것 같다.)
