

# Nuitrack Body Tracker

### for DeepTasK 인식 시스템  사용 가이드

## Description
NuiTracker ROS 포팅 버전, 사람 인식 및 위치 검출, 관절 정보 출력


## 프로젝트 환경

- OS : Ubuntu 16.04(**kernel 4.15**) 상위 버전 호환 가능
- ROS Version : Kinetic (ROS1), **ROS 선행 설치 필요**( Openni package dependency)


## 설치에 필요한 패키지
- 1) Realsense SDK 2.0 : v2.23  (링크 이용 설치 [librealsense](https://github.com/IntelRealSense/librealsense/blob/master/doc/distribution_linux.md#linux-distribution))
  - 1-1) 센서 호환 펌웨어 :  센서 하드웨어**D415 권장**(D435) v5.10.06
[센서 펌웨어 업데이트 참고](`https://www.intel.co.kr/content/www/kr/ko/support/articles/000028171/emerging-technologies/intel-realsense-technology.html ` )
- 2) Opencv (**v.2.4.13**)
- 3) Nuitracker : Driver(**0.23**), SDK(1.3.10 이하 버전) 최신 버전 동작 안할 수 있음



### 1) Realsense SDK 2.0 

**Intel® RealSense™ SDK 2.0** provides installation packages in [`dpkg`](https://en.wikipedia.org/wiki/Dpkg) format for Ubuntu 16 LTS*. * The Realsense [DKMS](https://en.wikipedia.org/wiki/Dynamic_Kernel_Module_Support) kernel drivers package (`librealsense2-dkms`) supports Ubuntu LTS kernels 4.4, 4.10 and 4.13.

#### Installing the packages:

- Add Intel server to the list of repositories :
  `echo 'deb http://realsense-hw-public.s3.amazonaws.com/Debian/apt-repo xenial main' | sudo tee /etc/apt/sources.list.d/realsense-public.list`
  It is recommended to backup `/etc/apt/sources.list.d/realsense-public.list` file in case of an upgrade.

- Register the server's public key :
  `sudo apt-key adv --keyserver keys.gnupg.net --recv-key 6F3EFCDE`

- Refresh the list of repositories and packages available :
  `sudo apt-get update`

- In order to run demos install:
  `sudo apt-get install librealsense2-dkms`
  `sudo apt-get install librealsense2-utils`
  The above two lines will deploy librealsense2 udev rules, kernel drivers, runtime library and executable demos and tools. Reconnect the Intel RealSense depth camera and run: `realsense-viewer`

  + realsense-viewer로 테스트 할 때 API-version mismatch가 발생할 경우
    `sudo realsense-viewer` 로 실행

  + Frames didn't arrived within 5 seconds 메시지 발생시
    RealSense HW 펌웨어 업데이트 필요[센서 펌웨어 업데이트](`https://www.intel.co.kr/content/www/kr/ko/support/articles/000028171/emerging-technologies/intel-realsense-technology.html `  참고 )

    업데이트 툴 설치 후 펌웨어 지시사항에 따라 펌웨어 업데이트 수행

- Developers shall install additional packages:
  `sudo apt-get install librealsense2-dev`
  `sudo apt-get install librealsense2-dbg`
  With `dev` package installed, you can compile an application with **librealsense** using `g++ -std=c++11 filename.cpp -lrealsense2` or an IDE of your choice.

  Verify that the kernel is updated :
  `modinfo uvcvideo | grep "version:"` should include `realsense` string



### 2) OpenCV 2.4

http://better-today.tistory.com/13 참조

#### Installing the packages:

- OpenCV prerequisites 를 설치.

```
$ sudo apt-get install libjpeg8-dev libtiff5-dev libjasper-dev libpng12-dev
$ sudo apt-get install libgtk2.0-dev
$ sudo apt-get install libavcodec-dev libavformat-dev libswscale-dev libv4l-dev
$ sudo apt-get install libatlas-base-dev gfortran
```

- OpenCV 를 다운받고 빌드한다.

```
$ git clone --branch 2.4 https://github.com/opencv/opencv.git
$ cd opencv
$ mkdir build
$ cd build 
$ cmake -D CMAKE_BUILD_TYPE=RELEASE \
    -D CMAKE_INSTALL_PREFIX=/usr/local \
    -D WITH_CUDA=OFF \
    -D BUILD_NEW_PYTHON_SUPPORT=ON \
    -D INSTALL_C_EXAMPLES=ON \
    -D INSTALL_PYTHON_EXAMPLES=ON \
    -D BUILD_EXAMPLES=ON ..
$ make -j8
$ sudo make install
$ sudo /bin/bash -c 'echo "/usr/local/lib" > /etc/ld.so.conf.d/opencv.conf'
$ sudo ldconfig
```



### 3-1)  Nuitrack Linux Driver

- Download Nuitrack from [here](http://download.3divi.com/Nuitrack/platforms/)

- Install the downloaded package using the following command:

  `sudo apt-get purge --auto-remove openni-utils`

  `sudo dpkg -i <downloaded-package-name>.deb`

- Check that the environment variables **NUITRACK_HOME** and **LD_LIBRARY_PATH** are set correctly using the following commands:

  `echo $NUITRACK_HOME`

  `echo $LD_LIBRARY_PATH`

  - If the environment variables are empty, set them manually using the following commands (as root):

    `echo "export NUITRACK_HOME=/usr/etc/nuitrack" > /etc/profile.d/nuitrack_env.sh`

    `echo "export LD_LIBRARY_PATH={$LD_LIBRARY_PATH}:/usr/local/lib/nuitrack" >> /etc/profile.d/nuitrack_env.sh`
필요시 파일 확인 후 {$LD_LIBRARY_PATH} 수동 추가(기존 Path 유지 위해)

    `. /etc/profile.d/nuitrack_env.sh`

    사용자 계정에서 위 명령어 실행

  - If you see "ERROR: Couldn't open device ..." message when trying to use Nuitrack, try to set permissions for USB devices with the following command: 

    `sudo chmod -R 777 /dev/bus/usb/`

- Open terminal and run command: `sudo -E nuitrack_license_tool` 

- Click "Compatibility test" and wait for it to complete 
  테스트가 동작하지 않을 때, opencv 재설치 및 PATH 확인(NUITRACK_HOME, LD_LIBRARY_PATH)
  ```ldconfig```  다시 쳐보기
  혹은 3D 센서 연결 확인(**USB 3.2**) 인식 필수
- Enter your secret key and press "Get Available Licenses" 
  **라이센스 관리 계정 : deeptask2017@gmail.com
  NAS\Common\Util\SW_라이센스_키 참고**
- Choose license type and press "Upgrade to Pro" or "Upgrade to Pro for 1 year" 

- Your device will get license from activation server and after that Nuitrack will be fully-functional on this device (without 3 minutes time restriction) 

- To test Nuitrack middleware press "Test" button 



### 3-2) Nuitrack Linux SDK

- Download Nuitrack from [Official](http://download.3divi.com/Nuitrack/)
- 1.3.10 버전은 [여기](https://drive.google.com/file/d/1P5PJgByVV51b9ND54ICGv4ZoBSO1wZLy/view?usp=sharing)
- Extract .zip file and copy to $(package_home) directory



## NuiTrack Body Tracker

This is a ROS Node to provide functionality of the NuiTrack SDK ([https://nuitrack.com](https://nuitrack.com/))

- NuiTrack is NOT FREE, but reasonably inexpensive, and works with a number of cameras (see their webpage)
- Also see Orbbec Astra SDK as a possible alternative (but only for Orbbec cameras)

Publishes 3 messages:

1. body_tracking_position_pub_ custom message: <body_tracker_msgs::BodyTracker> Includes: 2D position of person relative to head camera; allows for fast, smooth tracking Joint.proj: position in normalized projective coordinates (x, y from 0.0 to 1.0, z is real) Astra Mini FOV: 60 horz, 49.5 vert (degrees)

3D position of the shoulder joint in relation to robot (using TF) Joint.real: position in real world coordinates Useful for tracking person in 3D

1. body_tracking_skeleton_pub_ custom message: <body_tracker_msgs::Skeleton> Includes: Everyting in BodyTracker message above, plus 3D position of upper body joints in relation to robot (using TF)
2. marker_pub_ message: <visualization_msgs::Marker> Publishes 3d markers for selected joints. Visible as markers in RVIZ.

### Installation
- **상위 프로젝트 git 이용 설치시 다음 repository 이용X**
	- Clone [body_tracker_msgs](https://github.com/shinselrobots/body_tracker_msgs) into your Catkin workspace
  - `git clone https://github.com/shinselrobots/body_tracker_msgs.git `
  - catkin_make to confirm complies OK
	- Clone [this project](https://github.com/shinselrobots/nuitrack_body_tracker) into your Catkin workspace
  - `git clone https://github.com/shinselrobots/nuitrack_body_tracker.git `
  - catkin_make to confirm complies OK

### Test Nuitrack SDK

- REBOOT! If you run into errors, it is probably becuse you did not reboot.
- Follow instructions at: ~/NuitrackSDK/Examples/nuitrack_gl_sample/README.txt

```
=============================================================
	Ubuntu
=============================================================

Dependencies:

	sudo apt-get install freeglut3-dev

Dependencies for Ubuntu 14.04:

	sudo apt-get install libgl1-mesa-dri

Build sample:

	cd <path_to_NuitrackSDK_folder>/Examples/nuitrack_gl_sample
	mkdir build
	cd build
	cmake ..
	make


Run sample:
	./nuitrack_gl_sample [path/to/nuitrack.config]
```
- 카메라 재설치 후 모듈 자체 테스트시 위 샘플 프로그램 사용
-  /usr/etc/nuitrack/data/nuitrack.config
nuitracker 성능 Tunning 을 위한 파라미터 조절 가능


### Running nuitrack_body_tracker node

+ (중요!) CMakeList.txt 수정(NUITRACK_SDK_PATH)

  ```
  # For nuitrack
  # add_compile_options(-std=c++11 -D_GLIBCXX_USE_CXX11_ABI=0)
  set(NUITRACK_SDK_PATH /home/bluish02/repository/DeepTasK_Kist_NUI/NuitrackSDK) 
  ```

+ ` roslaunch nuitrack_body_tracker nuitrack_body_tracker.launch `

### Todo List
- NUI SDK 최신 버전 호환X -> opencv3.X 설치 및 확인 필요
- Skeleton marker 사용한 3D 시각화 -> RVIZ 이용
- Ground or Moving Object 오인식 문제 -> 
- rosbag을 이용한 모듈 실행

#### contributer
2018년 8월 8일  최태민(choitm0707@kist.re.kr)
2019년 12월 05일  이겨레(lkrrufp@kist.re.kr)