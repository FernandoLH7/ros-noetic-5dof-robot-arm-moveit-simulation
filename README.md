# ROS Noetic 5DOF Robot Arm Grasp Simulation

<p align="center">
  <img src="media/video_urdf5dofmoveit_preview.gif" alt="Funcionamiento del brazo robotico 5DOF en Gazebo y MoveIt" width="570">
</p>

<p align="center">
  <a href="media/video_urdf5dofmoveit_sin_audio.mp4">Ver video completo</a>
</p>

Simulación en ROS Noetic de un brazo robótico de 5 grados de libertad con gripper, integración en Gazebo, visualización en RViz y planeación de movimiento con MoveIt. El workspace permite cargar el modelo URDF del robot, inicializar controladores ROS, ejecutar MoveIt y validar rutinas de movimiento predefinidas para una escena de agarre.

El repositorio contiene únicamente el código fuente necesario para reconstruir el workspace. Los directorios generados por compilación, como `build/`, `devel/`, `logs/` y `.catkin_tools/`, no se versionan porque deben crearse localmente con `catkin build`.

## Estado Del Proyecto

- Distribución objetivo: ROS Noetic sobre Ubuntu 20.04.
- Simulador: Gazebo 11 con plugins de soporte para agarre.
- Planeación: MoveIt con configuración del paquete `arm_robot_moveit`.
- Modelo principal: `robot_arm_urdf` con mallas STL y controladores ROS.
- Mundo principal: `object2.world`, corregido para evitar duplicar el modelo `robot_arm_urdf` dentro del archivo de mundo. El robot se inserta desde el launch mediante `spawn_model` y la descripción URDF.

## Estructura

```text
.
├── LICENSE
├── README.md
└── src
    ├── CMakeLists.txt
    ├── arm_robot_moveit
    │   ├── config
    │   ├── launch
    │   ├── Objects
    │   ├── scripts
    │   └── worlds
    ├── gazebo-pkgs
    │   ├── gazebo_grasp_plugin
    │   ├── gazebo_grasp_plugin_ros
    │   ├── gazebo_state_plugins
    │   ├── gazebo_test_tools
    │   ├── gazebo_version_helpers
    │   └── gazebo_world_plugin_loader
    ├── general-message-pkgs
    │   ├── object_msgs
    │   ├── object_msgs_tools
    │   └── path_navigation_msgs
    └── robot_arm_urdf
        ├── config
        ├── launch
        ├── meshes
        └── urdf
```

## Requisitos

Usa Ubuntu 20.04 con ROS Noetic instalado y una terminal Bash con el entorno de ROS cargado.

```bash
sudo apt update
sudo apt install -y \
  python3-catkin-tools python3-rosdep \
  gazebo11 \
  ros-noetic-moveit \
  ros-noetic-gazebo-ros-pkgs \
  ros-noetic-gazebo-ros-control \
  ros-noetic-ros-controllers \
  ros-noetic-joint-state-controller \
  ros-noetic-joint-trajectory-controller \
  ros-noetic-position-controllers \
  ros-noetic-joint-state-publisher \
  ros-noetic-joint-state-publisher-gui \
  ros-noetic-robot-state-publisher \
  ros-noetic-xacro \
  ros-noetic-rviz \
  ros-noetic-object-recognition-msgs \
  ros-noetic-eigen-conversions
```

Si `rosdep` no está inicializado en tu instalación:

```bash
sudo rosdep init
rosdep update
```

## Clonar Y Compilar

Clona el repositorio como raíz de un workspace catkin. El repositorio ya incluye la carpeta `src`, por eso no debe clonarse dentro de otra carpeta `src`.

```bash
mkdir -p ~/robot_arm_ws
cd ~/robot_arm_ws
git clone https://github.com/FernandoLH7/ros-noetic-5dof-robot-arm-grasp-simulation.git .

source /opt/ros/noetic/setup.bash
rosdep install --from-paths src --ignore-src -r -y
catkin config --extend /opt/ros/noetic
catkin build
source devel/setup.bash
```

Para reconstruir desde cero:

```bash
cd ~/robot_arm_ws
catkin clean -y
rm -rf build devel logs .catkin_tools
source /opt/ros/noetic/setup.bash
catkin config --extend /opt/ros/noetic
catkin build
source devel/setup.bash
```

## Plugin De Agarre En Gazebo

El plugin `gazebo_grasp_plugin` genera la biblioteca `libgazebo_grasp_fix.so`. Después de compilar, valida que exista:

```bash
cd ~/robot_arm_ws
ls devel/lib/libgazebo_grasp_fix.so
```

Antes de iniciar Gazebo, agrega la ruta de plugins generada por el workspace:

```bash
export GAZEBO_PLUGIN_PATH="$GAZEBO_PLUGIN_PATH:$(pwd)/devel/lib"
```

Para dejarlo permanente en tu usuario:

```bash
echo 'export GAZEBO_PLUGIN_PATH="$GAZEBO_PLUGIN_PATH:$HOME/robot_arm_ws/devel/lib"' >> ~/.bashrc
source ~/.bashrc
```

Durante el arranque, Gazebo debe poder cargar el plugin sin errores de biblioteca no encontrada. Los mensajes de validación del gripper suelen aparecer en la terminal donde se ejecutó `roslaunch`.

## Ejecutar La Simulación

En una terminal:

```bash
cd ~/robot_arm_ws
source /opt/ros/noetic/setup.bash
source devel/setup.bash
export GAZEBO_PLUGIN_PATH="$GAZEBO_PLUGIN_PATH:$(pwd)/devel/lib"
roslaunch arm_robot_moveit robot_arm_sim.launch
```

Este launch inicializa Gazebo con `object2.world`, publica el URDF del robot, inserta el modelo `robot_arm_urdf`, carga controladores ROS, arranca `move_group` y abre RViz con la configuración de MoveIt.

## Ejecutar Una Pose Predefinida

Con la simulación activa, abre otra terminal:

```bash
cd ~/robot_arm_ws
source /opt/ros/noetic/setup.bash
source devel/setup.bash
chmod +x src/arm_robot_moveit/scripts/node_set_predefined_pose.py
rosrun arm_robot_moveit node_set_predefined_pose.py
```

El script utiliza MoveIt para enviar el brazo a una pose definida en el código y permite validar el flujo de planeación y ejecución.

## Solución De Problemas

### Error: el modelo `robot_arm_urdf` ya existe

Si Gazebo muestra un error indicando que el nombre del modelo ya existe, significa que el robot se insertó dos veces: una desde el mundo y otra desde el launch. En este repositorio, `object2.world` queda preparado para no incluir el modelo completo del robot; el robot se debe cargar desde `robot_arm_urdf/launch/arm_urdf.launch`.

### Mallas O Modelos Con Rutas Locales

Los archivos del repositorio no deben depender de rutas absolutas como `/home/<usuario>/...`. Si agregas nuevos objetos al mundo, usa rutas portables de ROS o Gazebo, por ejemplo `package://...` o `model://...`.

### Warnings De PID En Controladores

Gazebo puede mostrar advertencias de ganancias PID faltantes si algún controlador no define todos sus parámetros. No siempre bloquean la simulación, pero si un joint no responde, revisa `robot_arm_urdf/config/joint_trajectory_controller.yaml` y la configuración de controladores usada por `arm_urdf.launch`.

### El Plugin Del Gripper No Carga

Verifica estos puntos:

```bash
cd ~/robot_arm_ws
source devel/setup.bash
ls devel/lib/libgazebo_grasp_fix.so
echo $GAZEBO_PLUGIN_PATH
```

Si la biblioteca no existe, recompila con `catkin build`. Si existe pero Gazebo no la encuentra, exporta de nuevo `GAZEBO_PLUGIN_PATH` apuntando a `~/robot_arm_ws/devel/lib`.

### Cerrar ROS Y Gazebo

Si quedan procesos abiertos después de cerrar ventanas:

```bash
pkill -f gzserver
pkill -f gzclient
pkill -f rosmaster
pkill -f rosout
```

## Notas De Mantenimiento

- Mantén fuera del repositorio cualquier salida generada localmente por compilación o ejecución.
- Después de modificar URDF, controladores o plugins, recompila con `catkin build` y vuelve a cargar `devel/setup.bash`.
- Si se agregan mundos nuevos, evita incluir el robot completo dentro del `.world`; usa los launch files para cargar el URDF y conservar un único origen del modelo.
