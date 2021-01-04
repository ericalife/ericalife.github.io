---
title:  "Multi-UAV Control Project (1)"
excerpt: "편대비행 시뮬레이션을 해보자"

categories:
  - Blog
tags:
  - Blog
last_modified_at: 2020-12-28
---

편대비행 시뮬레이션 제작기를 글로 정리한다.

이유는 정리의 필요성을 느꼈기 때문이다..
수많은 양의 자료들을 통합하려면 이점이 훨씬 나을 것같다.
수많은 ppt들은 어느 컴퓨터에 있는지 어느 노트북에 있는지 정리하는데만 한 세월이다.
웹상에서 글이라도 적으면 좀 낫지않을까?에서 시작한다.


### 1. path 설정

```bash
echo "export GAZEBO_MODEL_PATH=${GAZEBO_MODEL_PATH}:$HOME/catkin_ws/src/iq_sim/models" >> ~/.bashrc
```

### 2. 편대 비행을 위한 드론을 여러개 만든다. 

```bash

~/catkin_ws/src/iq_sim/models$ #이 경로에서 drone모델을 drone2,drone3,... 복붙
```
    model 폴더에서 각 drone들의 특성 수정

```html
#model_config
<name>drone2</name> 
```
```html
#model_sdf
 <fdm_port_in>9012</fdm_port_in> 
   <fdm_port_out>9013</fdm_port_out> #drone2 : 9012-9013, drone3:9022-9023
 ```

### 3.  world 수정

```bash
~/catkin_ws/src/iq_sim/worlds$ sbl runway.world
```

```html
 <model name="drone2">
    <pose>10 0 0 0 0 0</pose>
     <include>
        <uri>model://drone2</uri>
      </include>
    </model>
```

### 4. launch the world

```bash
roslaunch iq_sim runway.launch 
```
![스크린샷, 2020-12-28 23-17-24](https://user-images.githubusercontent.com/76676102/103220965-dc820d80-4964-11eb-8196-73a64ebf9335.png)

launch하면 gazebo화면에서 드론 3대가 짠하고 나타난다.

현재는 각각의 드론별로 컨트롤러가 필요하다.
드론별로 컨트롤러를 연결하기 위해서는

```
sim_vehicle.py -v ArduCopter -f gazebo-iris --console -I0 #I1, I2,...
```

연결이 제대로 된지 확인해보려면 컨트롤러마다 다음의 명령어를 쳐준다.

![스크린샷, 2020-12-28 23-43-31](https://user-images.githubusercontent.com/76676102/103221677-8a41ec00-4966-11eb-937f-6d4d1a144ce5.png)

제대로 작성이 되었다면

드론이 하나씩 takeoff된다.

![스크린샷, 2020-12-28 23-45-58](https://user-images.githubusercontent.com/76676102/103221817-dee56700-4966-11eb-9391-b867ebf9a0ee.png)

### 5. QGroundControl을 사용해보자.


먼저 다음의 파일에 접근한다.

```bash
ardupilot/Tools/autotest/pysim/vehicleinfo.py
```

Gazebo-Iris부분을 다음과 같이 수정한다.
```
 "gazebo-drone1": {
                "waf_target": "bin/arducopter",
                "default_params_filename": ["default_params/copter.parm",
                                            "default_params/gazebo-drone1.parm"],
            },
            "gazebo-drone2": {
                "waf_target": "bin/arducopter",
                "default_params_filename": ["default_params/copter.parm",
                                            "default_params/gazebo-drone2.parm"],
            },
            "gazebo-drone3": {
                "waf_target": "bin/arducopter",
                "default_params_filename": ["default_params/copter.parm",
                                            "default_params/gazebo-drone3.parm"],
            },
```

드론 하나의 매개변수는 기본적으로 여기서 로드를 한다. 

다음으로 각각 매개변수 파일을 만든다.

```bash
ardupilot/Tools/autotest/default_params
```

```
default_params/gazebo-drone1.parm
default_params/gazebo-drone2.parm
default_params/gazebo-drone3.parm
```
맨 마지막 줄 SYSID_THISMAV만 변수에 맞게 1,2,3 등등으로 변경한다.
```
Example gazebo-drone1.param
# Iris is X frame
FRAME_CLASS 1
FRAME_TYPE  1
# IRLOCK FEATURE
RC8_OPTION 39
PLND_ENABLED    1
PLND_TYPE       3
# SONAR FOR IRLOCK
SIM_SONAR_SCALE 10
RNGFND1_TYPE 1
RNGFND1_SCALING 10
RNGFND1_PIN 0
RNGFND1_MAX_CM 5000
SYSID_THISMAV 1
```

### 6. QGroundControl과 드론 연결

QGroundControl.AppImage를 연결한 후

![스크린샷, 2020-12-29 00-20-13](https://user-images.githubusercontent.com/76676102/103224766-af852900-496b-11eb-9f2a-7e8e7b51a469.png)

각각의 드론의 통신링크를 설정한다. (port번호와 통신 타입 8100~)
![스크린샷, 2020-12-29 00-21-35](https://user-images.githubusercontent.com/76676102/103224862-ef4c1080-496b-11eb-8bcd-d6b527013646.png)

다시한번
```bash
roslaunch iq_sim runway.launch 
```

```
sim_vehicle.py -v ArduCopter -f gazebo-drone1 --console -I0 --out=tcpin:0.0.0.0:8100 
```
```
sim_vehicle.py -v ArduCopter -f gazebo-drone2 --console -I1 --out=tcpin:0.0.0.0:8200 
```

연결
![스크린샷, 2020-12-29 00-29-03](https://user-images.githubusercontent.com/76676102/103225265-e60f7380-496c-11eb-8219-8a98635560c9.png)

연결 후 이륙하면

![스크린샷, 2020-12-29 00-31-05](https://user-images.githubusercontent.com/76676102/103225403-2a9b0f00-496d-11eb-9ffe-2543dbcbf93d.png)


QGroundControl과 gazebo가 연결되어 드론이 이륙하는 모습을 볼 수 있다.
![스크린샷, 2020-12-29 00-31-41](https://user-images.githubusercontent.com/76676102/103225436-3f77a280-496d-11eb-917e-d151f57f1609.png)
