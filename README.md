# Lidar RS-Helios-16P
This repository describes the steps to initialize the Robosense Helios 16 lidar on a Linux server and how to get the cloud of points in Matlab using the ROS Toolbox.

## Requirements
* Ubuntu 18.04
* ROS Melodic desktop
* rslidar_sdk
* Matlab R2021b
* ROS Toolbox (Matlab)

***
## Setting the Lidar RS-Helios-16P
On the [Robosense official website](https://www.robosense.ai/en/resources-81) you can find  the Helios 16 manual, which is essential reading to understand how this device works. Sin embargo, aquí se muestran directamente los pasos a seguir para poner en marcha el RS-Helios-16P.

### 1. Configuración de la interfaz de red
1. Conectar el RS-Helios-16P al interface box y este al enchufe de electricidad y al computador mediante cable ethernet.
2. En Ubuntu, abre una terminal, localiza la interfaz de red a la cual has conectado el Lidar (escribe `sudo ip a`) y cofigura la IP como `192.168.1.102` con el siguiente comando, ya que por defecto el Helios 16 tiene la IP `192.168.1.200`.
~~~
sudo ifconfig <your_network_interface> 192.168.1.102
~~~

### 2. Configurar un servidor PTP
**Precision Time Protocol** (PTP) is a time synchronization protocol used for high-precision time synchronization between devices, which can meet higher precision time synchronization requirements and reach sub-microsecond level accuracy. Para configurar un servidor PTP en Linux, ejecuta los siguientes comandos:
1. Install ptpd:
~~~
sudo apt update
sudo apt install ptpd
~~~

2. Ejecuta ptpd como servidor (Master) en el puerto Ethernet al que has conectado el Lidar:
~~~
sudo ptpd -M -i <your_network_interface> -D
~~~
3. Verifica que funciona (deberías ver el estado **active (exited)**):ejecuta el segundo comando y vuelve a verificar con el primero
~~~
sudo service ptpd status
~~~
4. Si no está activo, entonces corre:
~~~
sudo service ptpd start
~~~

### 3. Descarga e instala ROS
**Robot Operating System** (ROS) es una plataforma de código abierto que facilita el desarrollo de aplicaciones robóticas. En la Web oficial de ROS se muestran detalladamente los [pasos para instalar ROS Melodic en Ubuntu 18.04](https://wiki.ros.org/melodic/Installation/Ubuntu).

> **NOTE:** *It's highly recommanded to install ros-distro-desktop-full. If you do so, the corresponding libraries, such as Yaml, PCL, will be installed at the same time.*

### 4. Descarga rslidar_sdk
rslidar_sdk is the Software Development Kit of the RoboSense Lidar based on Ubuntu. It allows you to get point cloud through ROS and contains the lidar driver core `rs_driver` which is essential for integrate the Lidar into your own projects. Haz clic en [este link](https://github.com/RoboSense-LiDAR/rslidar_sdk/tree/main?tab=readme-ov-file) para ir al repositorio oficial. Descarga la herramienta siguiendo los pasos que allí se describen.

1. Install libpcap (Essential)
~~~
sudo apt-get install -y  libpcap-dev
~~~

2. Crea una carpeta de trabajo (por ejemplo: **3dlidar**) en el user's home (i.e /home/user/). Luego accede a ella and create a src folder in it:
~~~
mkdir 3dlidar
cd 3dlidar/
mkdir src
~~~

3. Go to the src folder and clone the `rslidar_sdk` repository
~~~
cd src/
git clone https://github.com/RoboSense-LiDAR/rslidar_sdk.git
cd rslidar_sdk
git submodule init
git submodule update 
~~~

4. Go back to the workplace (`3dlidar/`), compile and run:
~~~
cd ..
catkin_make
source devel/setup.bash
roslaunch rslidar_sdk start.launch
~~~

### 4. Introduction to parameters
>**NOTE:** *this secion has beben taken from rslidar_sdk repository, for more detailed information check the [**Introduction to Parameters**](https://github.com/RoboSense-LiDAR/rslidar_sdk/blob/main/doc/intro/02_parameter_intro.md) section*

* rslidar_sdk reads parameters from the configuration file config.yaml, which is stored in rslidar_sdk/config. 

* config.yaml contains two parts, the common part and the lidar part.

> **NOTE:** *config.yaml is indentation sensitive! Please make sure the indentation is not changed after adjusting the parameters!* 

1. Go to the `rslidar_sdk/config` folder and open **config.yaml** file
~~~
cd 3dlidar/src/src/rslidar_sdk/config/
ls
sudo nano config.yaml
~~~

2. For the RS-Helios-16P choose the following options (leave the rest as default):
~~~
msg_source: 1 
send_packet_ros: false                               
send_point_cloud_ros: true 

lidar_type: RSHELIOS_16P

msop_port: 6699
difop_port: 7788
imu_port: 0
~~~

### 5. Capturing the cloud of point from Matlab





Helios 16 communicates with a computer via Ethernet using **User Datagram Protocol** (UDP) but the **Main Data Stream Output Protocol** (MSOP) encapsulates the laser scanning data,
including distance, angle, and reflection intensity, into packets for output.
