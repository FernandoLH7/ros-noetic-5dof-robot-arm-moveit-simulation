# general-message-pkgs

Colección de paquetes de mensajes reutilizados por los plugins y herramientas de simulación de este workspace.

## Dependencias

- `object_recognition_msgs`
- Paquetes locales de este repositorio dentro de `src/general-message-pkgs`

En ROS Noetic puedes instalar la dependencia externa con:

```bash
sudo apt install ros-noetic-object-recognition-msgs
```

## Compilación Dentro Del Workspace

Este repositorio ya incluye estos paquetes dentro de `src`, por lo que se compilan junto con el resto del proyecto:

```bash
cd ~/robot_arm_ws
source /opt/ros/noetic/setup.bash
catkin build
source devel/setup.bash
```
