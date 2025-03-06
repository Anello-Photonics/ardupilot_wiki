.. _common-external-ahrs-anellox3:

=======================
ANELLO Photonics X3
=======================

The ANELLO X3 is a compact and highly reliable optical gyroscope-based inertial measurement unit
(IMU) for autonomous applications in GNSS-denied environments. The X3 includes three ANELLO
SiPhOGTM sensors, each containing ANELLO’s high-performance single axis optical gyroscope and a 6-
axis MEMS IMU, along with a 3-axis magnetometer. The ANELLO X3 is the world’s smallest 3 axis optical
gyro-based IMU, with the low-noise (ARW < 0.05 deg/√hr) and low-drift (bias instability < 0.5 deg/hr)
required for autonomous applications in GNSS-denied environments. The low SWaP (size, weight, and
power) enables its use in small unmanned aerial and maritime vehicles.

This page describes how to configure, connect, and use the X3 within the Ardupilot ecosystem.


Hardware Setup
==============

Wiring
------

.. image:: ../../../images/anello_x3_wiring.png
    :target: ../_images/anello_x3_wiring.png


The X3 will work when connected to any of the available UARTs on your flight controller. Ensure that the port connected to is selected properly via the use of the SERIALx_PROTOCOL = 'AHRS' (36).


Sensor Configuration
====================

The X3 is capable of streaming data in both an ASCII format as well as an RCTM-based binary format. To communicate with Ardupilot, the system must be pre-configured to stream in RCTM mode.

To do this, the command ``#APCFG,W,mfm,4*7A`` must be passed to the X3. This can be done using any program which allows for serial data to be passed to the device, some examples including:

  - `Minicom <https://en.wikipedia.org/wiki/Minicom>`_
  - `PuTTY <https://www.putty.org/>`_





ArduPilot Configuration
=======================
There are two possible ways for VectorNav data to be used by ArduPilot: as an external sensor set to ArduPilot's EKFs or as an external AHRS. Both ways utilize ArduPilot's External AHRS driver.

To establish communication with the VectorNav unit, set the following for the relevant serial port:

  - ``SERIALx_PROTOCOL`` = 36 (AHRS)
  - ``SERIALx_BAUD`` = matching VectorNav sensor baudrate

.. tip::
  The External AHRS-specific parameters may not be visible before the ``Serialx_Protocol`` parameter is configured. As such, either a Refresh Params or a reset of ArduPilot may be necessary to see the parameters.

Use as an External Sensor Set
-----------------------------
If set up as an external sensor, VectorNav's raw sensor data (IMU, GNSS, Compass, Barometer, and GNSS, if available) can be used by ArduPilot's internal EKFs, as configured. After the serial parameters have been configured, configure:

- :ref:`AHRS_EKF_TYPE<AHRS_EKF_TYPE>` = 3 (ArduPilot’s EKF3)
- :ref:`EAHRS_TYPE<EAHRS_TYPE>` = 1 (VectorNav)
- :ref:`EAHRS_OPTIONS<EAHRS_OPTIONS>` bit 0 set to "1" value to disable ArduPilot's use of the bias-compensated IMU data, letting ArduPilot's filters do that (optional)
- :ref:`GPS1_TYPE<GPS1_TYPE>` = 21 (External AHRS) (If using a GNSS-enabled unit)
- :ref:`GPS2_TYPE<GPS2_TYPE>` = 21 (External AHRS) (If using a Dual GNSS-enabled unit)

If desired, :ref:`EAHRS_SENSORS<EAHRS_SENSORS>` may be used to specify which sensor data should be used by ArduPilot's filters.

Because ArduPilot's internal EKF will only update at the input IMU rate, it is recommended to raise the VectorNav IMU output rate beyond the default 50Hz. To do so, set :ref:`EAHRS_RATE<EAHRS_RATE>` to the desired IMU rate (800Hz maximum, 400Hz maximum for VN-300). Because the IMU output rate is configured on initialization, an ArduPilot reset is required after changing :ref:`EAHRS_RATE<EAHRS_RATE>`.

Use as an External AHRS
-----------------------
Configuring ArduPilot to use the VectorNav sensor as an External AHRS will use the VectorNav PVTA (position, velocity, time, and attitude) solution as canonical rather than one of the possible internal ArduPilot filters.
This will allow ArduPilot to use the VectorNav sensor's INS data that combines IMU and GNSS data in an advanced Kalman filtering estimation to provide position, velocity, and attitude estimates of higher accuracies and with better dynamic performance.

.. note::
  VectorNav uses the term AHRS to refer to an attitude-only solution, without absolute position measurement input. VectorNav uses the term INS to refer to a solution which accepts a position (often GNSS) measurement input and outputs a full PVTA. Because ArduPilot's External AHRS driver requires the data source (VectorNav) to provide an absolute PVT, use as an External AHRS is restricted to a VectorNav INS-enabled product (VN-2X0 or VN-3X0).

After the serial parameters have been configured, configure:
  - :ref:`AHRS_EKF_TYPE<AHRS_EKF_TYPE>` = 11 (External AHRS)
  - :ref:`EAHRS_TYPE<EAHRS_TYPE>` = 1 (VectorNAV)

.. tip::
  ArduPilot's internal navigation filters run even when configured to use a VectorNav as the canonical navigation source (unless internal filters are disabled). As such, it is recommended to additionally configure the VectorNav as an external sensor set. This allows ease of switching canonical PVTA between VectorNav's and ArduPilot's navigation filters, if necessary.
  To do this, configure the necessary paramters in Use as an External Sensor Set, but leave `AHRS_EKF_TYPE<AHRS_EKF_TYPE>` as External AHRS.

Published Data
==============

ArduPilot is configured to save VectorNav sensor data to a DataFlash Log as up to three messages: EAHI, EAHA, and EAHK.

The EAHI (External AHRS IMU) message contains IMU data outputs:

- Time (microseconds)
- Temperature (deg C)
- Pressure (Pa)
- Magnetometer (Gauss)
- Accelerometer (m/s^2)
- Gyroscope (rad/s)

The EAHA (External AHRS Attitude) message contains the following data outputs:

- Time (microseconds)
- Quaternion
- Yaw, pitch, roll (deg)
- Yaw, pitch, roll uncertainty (deg)

The EAHK (External AHRS INS/EKF) message contains INS data outputs:

- Time (microseconds)
- InsStatus
- Position LLA
- Velocity NED
- Position Uncertainty
- Velocity Uncertainty