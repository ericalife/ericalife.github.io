---
title:  "ROS message 마스터하기"
excerpt: "Publisher-Subscriber"

categories:
  - Blog
tags:
  - Blog
last_modified_at: 2021-01-06
---

## Service server, Client node

    서비스는 request가 있을 때만 response하는 service server와 request하고 response하는 service client로 나뉜다. 토픽과는 달리 일회성 메시지 통신이다. request, response가 끝나면 연결된 두 노드는 접속이 끝난다.
    

* service server
  ```bash
    add_executable(service_server src/service_server.cpp)
  ```
  <script src="https://gist.github.com/ericalife/803991f6a9a8b176bb9f83d0b5e1618c.js"></script>


* service client
  ```bash
    add_executable(service_client src/service_client.cpp)
  ```

  <script src="https://gist.github.com/ericalife/9e7b4b264a1a8faadf05f02f0312c6c5.js"></script>


  `실행`
  ```bash
    rosrun ros_tutorials_service service_server

    rosrun ros_tutorials_service service_client 2 3
  ```
  

출처 : [ROS 로봇프로그래밍 (표윤석)][link]

[link]: https://github.com/robotpilot/ros-seminar