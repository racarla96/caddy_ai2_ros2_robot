# caddy_ai2_ros2_robot

Paquete de integración para el carrito de golf eléctrico **Caddy AI2**, plataforma de pruebas de vehículos, en **hardware real**.  
Este repositorio es el punto de entrada para lanzar el sistema completo sobre el vehículo físico. No contiene lógica propia: orquesta todos los paquetes del ecosistema mediante launch files y configuración centralizada.

**ROS 2:** Jazzy | **Plataforma:** Ubuntu 24.04 | **Proyecto:** CERVAREC

---

## Arquitectura del sistema

```
[Nav2 / Planner]
        │  /cmd_vel  (geometry_msgs/Twist)
        ▼
[bicycle_to_ackermann_steering_adapter]   [bicycle_to_ackermann_traction_adapter]
        │  /forward_command_controller     │  /curtis_motor_velocity_controller
        │  /commands  (Float64MultiArray)  │  /commands  (Float64MultiArray)
        ▼                                  ▼
[steering_driver   ←─── ros2_control ───→ traction_driver]
  CANopen / can_steer_drv               CAN / can_motor_drv
        │                                  │
  Motor dirección + encoder abs.       Curtis motor driver
        └──────────────┬───────────────────┘
                       │  /joint_states
                       ▼
              [robot_state_publisher]
                       │  /tf
                       ▼
              [sbg_ig500n (IMU)]  +  [sick_lms_291 (LiDAR)]
```

---

## Repositorios dependientes

| Repositorio | Rama | Rol |
|---|---|---|
| `caddy_ai2_ros2_description` | `jazzy` | URDF / meshes del robot |
| `caddy_ai2_ros2_common` | `jazzy` | SocketCAN interface, launch utils |
| `caddy_ai2_ros2_control_system_steering_driver` | `jazzy` | HW interface dirección (CANopen CiA 402) |
| `caddy_ai2_ros2_control_system_traction_driver` | `jazzy` | HW interface tracción (Curtis CAN) |
| `caddy_ai2_ros2_bicycle_to_ackermann_steering_adapter` | `jazzy` | Conversión cinemática → steering |
| `caddy_ai2_ros2_bicycle_to_ackermann_traction_adapter` | `jazzy` | Conversión cinemática → traction |
| `caddy_ai2_ros2_control_sensors_sbg_ig-500n` | `jazzy` | Driver IMU SBG IG-500N |
| `caddy_ai2_ros2_sensors_sick_lms_291` | `jazzy` | Driver LiDAR SICK LMS 291 |

---

## Política de ramas

| Rama | Propósito |
|---|---|
| `jazzy` | Rama estable. Solo se integra código probado. |
| `feat/<nombre>` | Desarrollo de nuevas funcionalidades. Se abre desde `jazzy` y se integra con PR. Ejemplo: `feat/nav2-integration` |
| `hw/<nombre>` | Pruebas puntuales sobre hardware real que no son una feature completa. Vida corta. Ejemplo: `hw/calibracion-encoder-mayo` |

> **Regla general:** nunca se hace commit directo a `jazzy`. Todo entra por PR desde `feat/` o `hw/`.

---

## Instalación

### 1. Crear el workspace y clonar todos los repos

```bash
mkdir -p ~/ros2_ws/src && cd ~/ros2_ws/src

# Repos de integración
git clone -b jazzy https://github.com/racarla96/caddy_ai2_ros2_robot.git

# Descripción
git clone -b jazzy https://github.com/racarla96/caddy_ai2_ros2_description.git

# Librería común
git clone -b jazzy https://github.com/racarla96/caddy_ai2_ros2_common.git

# Drivers de actuadores
git clone -b jazzy https://github.com/racarla96/caddy_ai2_ros2_control_system_steering_driver.git
git clone -b jazzy https://github.com/racarla96/caddy_ai2_ros2_control_system_traction_driver.git

# Adaptadores cinemáticos
git clone -b jazzy https://github.com/racarla96/caddy_ai2_ros2_bicycle_to_ackermann_steering_adapter.git
git clone -b jazzy https://github.com/racarla96/caddy_ai2_ros2_bicycle_to_ackermann_traction_adapter.git

# Sensores
git clone -b jazzy https://github.com/racarla96/caddy_ai2_ros2_control_sensors_sbg_ig-500n.git
git clone -b jazzy https://github.com/racarla96/caddy_ai2_ros2_sensors_sick_lms_291.git
```

### 2. Instalar dependencias del sistema

```bash
# ROS 2 control
sudo apt install ros-jazzy-ros2-control ros-jazzy-ros2-controllers

# CAN tools y librerías
sudo apt install can-utils libsocketcan-dev

# Descripción del robot
sudo apt install ros-jazzy-xacro ros-jazzy-robot-state-publisher

# Sensores
sudo apt install ros-jazzy-imu-tools
```

### 3. Configurar interfaces CAN

Conectar ambos adaptadores USB-CAN y ejecutar los scripts incluidos en cada driver:

```bash
# Dirección (asigna nombre persistente can_steer_drv)
cd ~/ros2_ws/src/caddy_ai2_ros2_control_system_steering_driver/scripts
sudo ./setup_can_steer_drv.sh

# Tracción (asigna nombre persistente can_motor_drv)
cd ~/ros2_ws/src/caddy_ai2_ros2_control_system_traction_driver/scripts
sudo ./asignar_can_motor_drv.sh
```

### 4. Compilar

```bash
cd ~/ros2_ws
colcon build --symlink-install
source install/setup.bash
```

---

## Uso

### Lanzar el sistema completo

```bash
ros2 launch caddy_ai2_ros2_robot robot.launch.py
```

### Verificar que los controladores están activos

```bash
ros2 control list_controllers
ros2 control list_hardware_interfaces
```

### Monitorear buses CAN

```bash
# Terminal 1 - dirección
candump can_steer_drv

# Terminal 2 - tracción
candump can_motor_drv
```

### Enviar comando manual de prueba

```bash
# Velocidad lineal (m/s) y angular (rad/s)
ros2 topic pub /cmd_vel geometry_msgs/msg/Twist \
  "{linear: {x: 0.3}, angular: {z: 0.0}}" --once
```

---

## TODO

### Rama estable (`jazzy`)

#### Unificación de repositorios
- [ ] Alinear todas las ramas de los repos dependientes a `jazzy`
  - [ ] `caddy_ai2_ros2_control_system_steering_driver`: está en `jazzy-dev` → renombrar/mergear a `jazzy`
  - [ ] `caddy_ai2_ros2_description`: está en `jazzy-dev` → renombrar/mergear a `jazzy`
  - [ ] `caddy_ai2_ros2_mvsim_simulation`: está en `main` → migrar a `jazzy`
- [ ] Actualizar el README de `caddy_ai2_ros2_description` para que refleje la estructura multirepo actual (actualmente documenta un monorepo obsoleto)
- [ ] Eliminar las carpetas `bringup/` y `description/urdf` completos de dentro de `_steering_driver` y `_traction_driver`; dejar solo el fragmento `ros2_control.urdf.xacro` que describe el hardware interface
- [ ] Mover la configuración de controladores (`bringup/config/*.yaml`) de los repos de drivers a este repo

#### Launch system
- [ ] Crear `launch/robot.launch.py` — launch principal del sistema completo
  - [ ] Incluir `robot_state_publisher` con el URDF de `_description`
  - [ ] Incluir `controller_manager` con los dos hardware interfaces (steering + traction)
  - [ ] Incluir los dos nodos adaptadores bicycle→ackermann
  - [ ] Incluir driver IMU (`sbg_ig-500n`)
  - [ ] Incluir driver LiDAR (`sick_lms_291`)
- [ ] Añadir argumento `use_rviz` (default: false en robot real)
- [ ] Añadir argumento `use_joy` para activar control manual con joystick

#### Configuración centralizada
- [ ] Crear `config/controllers.yaml` unificado con todos los controladores
- [ ] Crear `config/robot_params.yaml` con parámetros globales (frecuencias, interfaces CAN, etc.)
- [ ] Documentar el procedimiento de calibración de `counts_per_radian` del encoder de dirección

#### Hardware y puesta en marcha
- [ ] Verificar y documentar el procedimiento completo de arranque del robot
- [ ] Añadir script de comprobación pre-vuelo (`scripts/check_hardware.sh`) que verifique:
  - [ ] Interfaces CAN activas (`can_steer_drv`, `can_motor_drv`)
  - [ ] IMU respondiendo
  - [ ] LiDAR respondiendo
- [ ] Documentar configuración de `/etc/sudoers` para interfaces CAN sin contraseña
- [ ] Estudiar configuración de kernel RT-PREEMPT para el PC embarcado
- [ ] Cambiar los drivers de tipo `System` a tipo `Actuator` en `_steering_driver` (TODO pendiente del propio repo)
- [ ] Añadir parámetro de interfaz CAN en el launch file de `_traction_driver` (TODO pendiente del propio repo)

#### Seguridad
- [ ] Implementar nodo de parada de emergencia (`/e_stop`)
- [ ] Definir comportamiento safe-state al perder comunicación CAN

---

## Estructura del paquete (objetivo)

```
caddy_ai2_ros2_robot/
├── config/
│   ├── controllers.yaml        # Configuración unificada de todos los controladores
│   └── robot_params.yaml       # Parámetros globales del sistema
├── launch/
│   ├── robot.launch.py         # Launch principal — sistema completo
│   ├── drivers.launch.py       # Solo actuadores (sin sensores)
│   └── sensors.launch.py       # Solo sensores
├── scripts/
│   └── check_hardware.sh       # Verificación pre-arranque
├── rviz/
│   └── robot.rviz              # Configuración de visualización
├── CMakeLists.txt
├── package.xml
└── README.md
```

---

## Licencia

Copyright (c) 2026, Rafael Carbonell Lázaro (racarla96) — CC BY 4.0
