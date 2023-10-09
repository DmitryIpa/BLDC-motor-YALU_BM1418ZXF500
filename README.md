# BLDC-motor-YALU_BM1418ZXF500
ODrive configuration for BLDC-motor YALU BM1418ZXF 500.
This repository contains a procedure for setting up an odrive, as well as an example of how to control an odrive.
![Image alt](https://github.com/DmitryIpa/Odrive_control/blob/main/odrive.png)
Версия odrivetool v0.6.7
## ИНСТРУКЦИЯ ПО НАСТРОЙКЕ ODrive v3.6
## В случае затруднений можно обратиться к [документации](https://docs.odriverobotics.com/v/latest/fibre_types/com_odriverobotics_ODrive.html#):
## Внимание! Перед использованием конфигурации ниже нужно проверить, чтобы версия ODrive была 0.5.6 или выше. Инструкция по прошивке ODrive находится [здесь](https://docs.odriverobotics.com/v/0.5.4/odrivetool.html?highlight=update#flashing-with-an-stlink) или на [github](https://github.com/DmitryIpa/ODrive-firmvare-update/blob/main/README.md)
# 1. Запускаем команду ниже для того, чтобы сбросить ODrive до настроек по умолчанию:
  ```
odrivetool
odrv0.erase_configuration()
  ```
# 2. Задаём параметры моторов для калибровки:
  ```
odrv0.axis0.motor.config.current_control_bandwidth = 2000
odrv0.axis0.motor.config.pole_pairs = 8
odrv0.axis0.motor.config.resistance_calib_max_voltage = 27
odrv0.axis0.motor.config.calibration_current = 12
odrv0.axis0.motor.config.current_lim_margin = 15 #40
odrv0.axis0.motor.config.requested_current_range = 60
odrv0.axis0.motor.config.torque_lim = "inf"
odrv0.axis0.motor.config.torque_constant = 0.761194029851
odrv0.axis0.motor.config.motor_type = MotorType.HIGH_CURRENT
odrv0.axis0.motor.config.current_lim = 13

odrv0.axis0.encoder.config.bandwidth = 2000
odrv0.axis0.encoder.config.mode = ENCODER_MODE_HALL
odrv0.axis0.encoder.config.cpr = 8 * 6
odrv0.axis0.encoder.config.calib_scan_distance = 90
  ```
# 3. Настройка режима пинов энкодера:
```

odrv0.config.gpio9_mode = GPIO_MODE_DIGITAL
odrv0.config.gpio10_mode = GPIO_MODE_DIGITAL
odrv0.config.gpio11_mode = GPIO_MODE_DIGITAL
odrv0.config.gpio12_mode = GPIO_MODE_DIGITAL
odrv0.config.gpio13_mode = GPIO_MODE_DIGITAL
odrv0.config.gpio14_mode = GPIO_MODE_DIGITAL
```
# 4. Настройка тормозного резистора. Включаем РЕЗИСТОР и задаём ему сопротивление в Омах:
```
odrv0.config.enable_brake_resistor = True
odrv0.config.brake_resistance = 1.2
```
# 5. Контроллер. В зависимости от ситуации изменяем параметры ПИ-регулятора. Ниже по умолчанию:
```

odrv0.axis0.controller.config.pos_gain = 30
odrv0.axis0.controller.config.vel_gain = 20
odrv0.axis0.controller.config.vel_integrator_gain = 35
odrv0.axis0.controller.config.vel_limit = "inf"
```

# 6. Сохраняем все заданные параметры
```

odrv0.save_configuration()
```

# 7. Этап калибровки. 
  1) Проверяем, чтобы параметры ниже имели значение False:
  ```
  odrv0.axis0.motor.config.pre_calibrated
  odrv0.axis0.encoder.config.pre_calibrated
  ```
  2) Самый быстрый и простой способ калибровки мотора и энкодера:
  ```
  odrv0.axis0.requested_state = AXIS_STATE_FULL_CALIBRATION_SEQUENCE
  ```
  Что происходит? На блоке питания повышается ток. Далее должен быть звуковой сигнал ODrive, после чего двигатель начинает вращаться. Ждем, когда он повернется в обе стороны. Если двигатель замер в процессе вращения, или не подал звуковой сигнал, то используем команду ниже для проверки ошибок:
  ```
  dump_errors(odrv0)
  ```
    
  Если вы - молодец, то драйвер не выдаст ошибки в консоли, ДА ЕЩЁ И РАССЧИТАЕТ ПАРАМЕТРЫ СОПРОТИВЛЕНИЯ ДВИГАТЕЛЯ И СМЕЩЕНИЯ ЭНКОДЕРА:
    ```
    odrv0.axis0.motor
    ```
  Найдите следующие строки:
  
  error = 0x0000 (int)
  phase_inductance = 0.00033594953129068017 (float)
  phase_resistance = 0.1793474406003952 (float)
  
  И для энкодера тоже:
    ```
    odrv0.axis0.encoder
    ```
  Найти параметр:
  phase_offset_float = 0.5126956701278687 (float)

# 8. Последний этап. Сохранение параметров калибровки
  1) Даём драйверу знать, что мотор и энкодер откалиброваны. Это требуется для того, чтобы при каждом запуске ODrive моторы не калибровались.
  ```
  odrv0.axis0.motor.config.pre_calibrated = True
  odrv0.axis0.encoder.config.pre_calibrated = True
  ```
  2) Сохраняем изменения
  ```
  odrv0.save_configuration()
  ```
# 9. Тестирование двигателей.
  1) Чтобы запустить двигатель, нужно ввести следующий набор параметров:


  Управляем в режиме позиционного управления:
  ```
  odrv0.axis1.requested_state = AXIS_STATE_CLOSED_LOOP_CONTROL
  odrv0.axis1.controller.config.control_mode = CONTROL_MODE_POSITION_CONTROL
  odrv0.axis1.controller.input_pos = 2
  ```

  Или запускаем двигатель на 20 оборотах в секунду:
  ```
  odrv0.axis0.requested_state = AXIS_STATE_CLOSED_LOOP_CONTROL
  odrv0.axis0.controller.config.control_mode = CONTROL_MODE_VELOCITY_CONTROL
  odrv0.axis0.controller.input_vel = 20
  ```
  Также можно задать управлять двигателем с помощью требуемого момента:
  
  ```
  odrv0.axis0.requested_state = AXIS_STATE_CLOSED_LOOP_CONTROL
  odrv0.axis0.controller.config.control_mode = CONTROL_MODE_TORQUE_CONTROL
  odrv0.axis0.controller.input_torque = 10.2
  ```
  Если нужно остановить:
  ```
  odrv0.axis0.controller.input_vel = 0
  odrv0.axis0.requested_state = AXIS_STATE_IDLE
  odrv0.axis1.controller.input_vel = 0
  odrv0.axis1.requested_state = AXIS_STATE_IDLE
  ```
  Во время управления двигателем на высоких скоростях может возникнуть ошибка 
    ```
          CURRENT_LIMIT_VIOLATION 
    ```
  В таком случае, нужно постепенно увеличивать параметр
    ```
          odrv0.axis0.motor.config.current_lim_margin
    ```
  Пока не пропадет ошибка. Или программно ограничить максимальное число оборотов двигателя.
  
  Если ещё что-то не так - перезагружаем ODrive:
  ```
  odrv0.reboot()
  ```
  Для сброса ошибок можно вызвать команду:
  ```
  odrv0.clear_errors()
  ```

# 10. Аналогично для мотора на M1
```
odrv0.axis1.motor.config.current_control_bandwidth = 2000
odrv0.axis1.motor.config.pole_pairs = 8

odrv0.axis1.motor.config.resistance_calib_max_voltage = 27

odrv0.axis1.motor.config.calibration_current = 12
odrv0.axis1.motor.config.current_lim_margin = 15 #40
odrv0.axis1.motor.config.current_lim = 13
odrv0.axis1.motor.config.requested_current_range = 60
odrv0.axis1.motor.config.torque_lim = "inf"
odrv0.axis1.motor.config.torque_constant = 0.761194029851
odrv0.axis1.motor.config.motor_type = MotorType.HIGH_CURRENT

odrv0.axis1.encoder.config.bandwidth = 2000
odrv0.axis1.encoder.config.mode = ENCODER_MODE_HALL
odrv0.axis1.encoder.config.cpr = 8 * 6
odrv0.axis1.encoder.config.calib_scan_distance = 90

odrv0.axis1.controller.config.pos_gain = 30
odrv0.axis1.controller.config.vel_gain = 20
odrv0.axis1.controller.config.vel_integrator_gain = 35
odrv0.axis1.controller.config.vel_limit = "inf"
```
Сохраняем параметры:
```
odrv0.save_configuration()
```
Калибруем и сохраняем состояние калибровки:
```
odrv0.axis1.requested_state = AXIS_STATE_FULL_CALIBRATION_SEQUENCE
```
```
odrv0.axis1.motor.config.pre_calibrated = True
odrv0.axis1.encoder.config.pre_calibrated = True
```
Сохраняем калибровку:
```
odrv0.save_configuration()
```

Управляем в режиме позиционного управления:
```
odrv0.axis1.requested_state = AXIS_STATE_CLOSED_LOOP_CONTROL
odrv0.axis1.controller.config.control_mode = CONTROL_MODE_POSITION_CONTROL
odrv0.axis1.controller.input_pos = 2
```
Или в режиме управления скоростью (20 об/с):
```
odrv0.axis1.requested_state = AXIS_STATE_CLOSED_LOOP_CONTROL
odrv0.axis1.controller.config.control_mode = CONTROL_MODE_VELOCITY_CONTROL
odrv0.axis1.controller.input_vel = 20
```
Или в режиме управления моментом:
```
  odrv0.axis0.requested_state = AXIS_STATE_CLOSED_LOOP_CONTROL
  odrv0.axis0.controller.config.control_mode = CONTROL_MODE_TORQUE_CONTROL
  odrv0.axis0.controller.input_torque = 10.2
```
Если нужно остановить:
```
odrv0.axis0.controller.input_vel = 0
odrv0.axis0.requested_state = AXIS_STATE_IDLE
odrv0.axis1.controller.input_vel = 0
odrv0.axis1.requested_state = AXIS_STATE_IDLE
```
