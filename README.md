# caddy_ai2_ros2_robot

Repositorio de integración del **Caddy AI2** (carrito de golf, plataforma de pruebas de vehículos, proyecto CERVAREC) en **hardware real**.
El vehículo lleva dos ordenadores, así que el sistema se divide en dos subrepos, cada uno un workspace ROS 2 con sus paquetes como submódulos:

| Submódulo | Ordenador | Contenido |
|---|---|---|
| [`lowlevel/`](lowlevel) | **LattePanda Alpha** | Control de bajo nivel: dirección, tracción, SICK LMS 291, SBG IG-500N (IMU/GPS), PLC y freno automático |
| [`perception/`](perception) | **Jetson Orin NX** (contenedor Docker) | Percepción: GPS u-blox, lidar 3D Helios RS-5515 y dos ZED 2i |

**ROS 2:** Jazzy | **Plataforma:** Ubuntu 24.04

## Clonar

```bash
git clone --recurse-submodules git@github.com:racarla96/caddy_ai2_ros2_robot.git
```

En cada ordenador basta con clonar y compilar solo el subrepo que le toca
(`caddy_ai2_ros2_robot_lowlevel` en el LattePanda, `caddy_ai2_ros2_robot_perception` en la Jetson).
Consulta el README de cada uno para compilar y lanzar.

## Historia

Hasta esta reorganización este repo era un único paquete de integración con un `.repos`. Ese paquete (launch y
configuración) vive ahora en `lowlevel/src/caddy_ai2_ros2_robot_lowlevel`.

Plan: [`PLAN_REORGANIZACION.md`](https://github.com/racarla96/caddy_ai2/blob/jazzy/docs/PLAN_REORGANIZACION.md).
