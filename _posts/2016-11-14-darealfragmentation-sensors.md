---
layout: post
title: Da Real Fragmentation - Sensors
summary: In this article, we are going to describe briefly how to use sensors in Android. We'll analyze how they behave under battery optimization modes and we'll explain some tricks in order to solve such fragmentation issues.
---

In the previous [post](http://pguardiola.com/blog/darealfragmentation-doze/), we walked through [Doze and App Standby](https://developer.android.com/training/monitoring-device-state/doze-standby.html) modes, we mentioned the different types of Doze and when they are activated. We showed all the restrictions that each bring (especially when working with Alarms) and how to dodge them. In this article, we are going to describe briefly how to use [sensors](https://developer.android.com/guide/topics/sensors/sensors_overview.html) in Android. We'll analyze how they behave under battery optimization modes and we'll explain some tricks in order to solve such fragmentation issues.

<!-- more -->

![Dragon-Ball-Radar](https://raw.githubusercontent.com/Guardiola31337/guardiola31337.github.io/master/art/darealfragmentation/sensors/dragon-ball-radar.gif)

## Sensors

Android _sensors_ are virtual devices that provide data coming from a set of physical sensors: accelerometers, gyroscopes, magnetometers, barometer, humidity, pressure, light, proximity and heart rate sensors. These _sensors_ are capable of providing raw data with high precision and accuracy, and are necessary if you want to set the brightness of your display automatically or you want to track device movement, for example.

Below is the list of the different categories of sensors that Android supports.

* **Motion** sensors measure acceleration forces and rotational forces along three axes. This category includes accelerometers, gravity sensors, gyroscopes, and rotational vector sensors.

* **Environmental** sensors measure various environmental parameters, such as ambient air temperature and pressure, illumination, and humidity. This category includes barometers, photometers, and thermometers.

* **Position** sensors measure the physical position of a device. This category includes orientation sensors and magnetometers.

### Types

The Android sensor framework lets you access many types of sensors.

* **Hardware-based** sensors are physical components built into a handset or tablet device. They derive their data from directly measuring specific environmental properties, such as acceleration, geomagnetic field strength, or angular change.

* **Software-based** sensors are not physical devices, although they mimic hardware-based sensors. They derive their data from one or more of the hardware-based sensors and are sometimes called virtual sensors or synthetic sensors. Examples include the linear acceleration sensor or the gravity sensor.

**Warning**: Not all devices will have all possible sensors (you will have to determine which ones are present) and a device can have more than one sensor of a given type. We will discuss the framework usage in a bit.

[Here](https://source.android.com/devices/sensors/sensor-types.html) you can find all sensor types supported by the Android platform.

**Warning**: While sensor availability varies from device to device, it can also vary between Android versions. So, if in doubt, check sensor availability by platform [here](https://developer.android.com/guide/topics/sensors/sensors_overview.html#table2).

### Framework

In order to use sensors in our application, Android provides several classes and interfaces that help you access these sensors and acquire raw sensor data and perform a wide variety of sensor-related tasks. Following we are going to describe them briefly. 

* [`SensorManager`](https://developer.android.com/reference/android/hardware/SensorManager.html): You can use this class to create an instance of the sensor service ([`Context.getSystemService()`](https://developer.android.com/reference/android/content/Context.html#getSystemService(java.lang.String)) with the argument [`SENSOR_SERVICE`](https://developer.android.com/reference/android/content/Context.html#SENSOR_SERVICE)). This class provides various methods for accessing ([`getDefaultSensor()`](https://developer.android.com/reference/android/hardware/SensorManager.html#getDefaultSensor(int))) and listing sensors ([`getSensorList()`](https://developer.android.com/reference/android/hardware/SensorManager.html#getSensorList(int))), registering ([`registerListener()`](https://developer.android.com/reference/android/hardware/SensorManager.html#registerListener(android.hardware.SensorEventListener, android.hardware.Sensor, int))) and unregistering ([`unregisterListener()`](https://developer.android.com/reference/android/hardware/SensorManager.html#unregisterListener(android.hardware.SensorEventListener))) sensor event listeners, and acquiring orientation information ([`getOrientation()`](https://developer.android.com/reference/android/hardware/SensorManager.html#getOrientation(float[], float[]))). This class also provides several sensor constants that are used to report sensor accuracy ([`SENSOR_STATUS_ACCURACY_LOW`](https://developer.android.com/reference/android/hardware/SensorManager.html#SENSOR_STATUS_ACCURACY_LOW), [`SENSOR_STATUS_ACCURACY_MEDIUM`](https://developer.android.com/reference/android/hardware/SensorManager.html#SENSOR_STATUS_ACCURACY_MEDIUM) and [`SENSOR_STATUS_ACCURACY_HIGH`](https://developer.android.com/reference/android/hardware/SensorManager.html#SENSOR_STATUS_ACCURACY_HIGH)), set data acquisition rates ([`SENSOR_DELAY_NORMAL`](https://developer.android.com/reference/android/hardware/SensorManager.html#SENSOR_DELAY_NORMAL), [`SENSOR_DELAY_UI`](https://developer.android.com/reference/android/hardware/SensorManager.html#SENSOR_DELAY_UI), [`SENSOR_DELAY_GAME`](https://developer.android.com/reference/android/hardware/SensorManager.html#SENSOR_DELAY_GAME) and [`SENSOR_DELAY_FASTEST`](https://developer.android.com/reference/android/hardware/SensorManager.html#SENSOR_DELAY_FASTEST)), and calibrate sensors.

* [`Sensor`](https://developer.android.com/reference/android/hardware/Sensor.html): You can use this class to create an instance of a specific sensor. This class provides various methods that let you determine a sensor's capabilities.

* [`SensorEvent`](https://developer.android.com/reference/android/hardware/SensorEvent.html): The system uses this class to create a sensor event object, which provides information about a sensor event. A sensor event object includes the following information: the raw sensor data ([`values`](https://developer.android.com/reference/android/hardware/SensorEvent.html#values)), the type of [`sensor`](https://developer.android.com/reference/android/hardware/SensorEvent.html#sensor) that generated the event, the [`accuracy`](https://developer.android.com/reference/android/hardware/SensorEvent.html#accuracy) of the data, and the [`timestamp`](https://developer.android.com/reference/android/hardware/SensorEvent.html#timestamp) for the event.

* [`SensorEventListener`](https://developer.android.com/reference/android/hardware/SensorEventListener.html): You can use this interface to create two callback methods that receive notifications (sensor events) when sensor values change ([`onSensorChanged`](https://developer.android.com/reference/android/hardware/SensorEventListener.html#onSensorChanged(android.hardware.SensorEvent))) or when sensor accuracy changes ([`onAccuracyChanged()`](https://developer.android.com/reference/android/hardware/SensorEventListener.html#onAccuracyChanged(android.hardware.Sensor, int))).

You can use the sensor framework to:

* Determine which sensors are available on a device.

* Determine an individual sensor's capabilities, such as its maximum range ([`getMaximumRange()`](https://developer.android.com/reference/android/hardware/Sensor.html#getMaximumRange())), manufacturer ([`getVendor()`](https://developer.android.com/reference/android/hardware/Sensor.html#getVendor())), power requirements ([`getPower()`](https://developer.android.com/reference/android/hardware/Sensor.html#getPower())), and resolution ([`getResolution()`](https://developer.android.com/reference/android/hardware/Sensor.html#getResolution())).

* Acquire raw sensor data and define the minimum rate ([`getMinDelay()`](https://developer.android.com/reference/android/hardware/Sensor.html#getMinDelay())) at which you acquire sensor data. If a sensor returns zero when you call the `getMinDelay()` method, it means the sensor is not a streaming sensor because it reports data only when there is a change in the parameters it is sensing.
  **Warning**: A sensor's maximum data acquisition rate is not necessarily the rate at which the sensor framework delivers sensor data to your application.

* Register and unregister sensor event listeners that monitor sensor changes.

### Batching

API level 19 added batched sensor events and you can specify a batch period in microseconds (`maxReportLatencyUs`), and events can be delayed before being reported to the application by up to that amount of time. As you can imagine, batching capabilities are an important consideration for power optimization.

**Warning**: Not all hardware supports this sort of batching behavior.

### Problems

Beware of the following snags:

* The delay that you specify is only a suggested delay. The Android system and other applications can alter this delay. There is no public method for determining the rate at which the sensor framework is sending sensor events to your application.
  **Warning**: If for some reason you do need to change the delay, you will have to unregister and reregister the sensor listener.

* The system will not disable sensors automatically unless you use __trigger__ sensors which were introduced in Android 4.4 and are designed to return a single sample and then automatically become unregistered.

* The sensor coordinate system is always based on the natural orientation of a device, which means that your application must not assume that a device's natural (default) orientation is portrait.
  **Warning**: Some sensors and methods use a coordinate system that is relative to the world's frame of reference. These sensors and methods return data that represent device motion or device position relative to the earth.

* The `SensorEvent` object received when `onSensorChanged()` is called comes from an object pool and gets recycled. So, it is not safe for you to hold onto this `SensorEvent` object **past** the call to `onSensorChanged()`.

### Recommendations

When working with sensors, you should take into account the guidelines described below:

* When possible, specify the largest delay that you can because the system typically uses a smaller delay. Using a larger delay imposes a lower load on the processor and therefore uses less power.

* You can use the timestamps that are associated with each sensor event to calculate the sampling rate over several events.

* You should always disable sensors you don't need, especially when your activity is paused. Failing to do so can drain the battery in just a few hours because some sensors have substantial power requirements and can use up battery power quickly.

* You should do as little as possible within the `onSensorChanged()` method so you don't block it.

* Avoid using deprecated methods or sensor types.

* Always verify that a sensor exists on a device before you attempt to acquire data from it.

### Conclusion

Nowadays, fragmentation in Android is a fact. As developers, we have to deal constantly with random behaviors between different versions of the operating system, incomplete ROMs, different hardware qualities, manufacturers that modify the specs of some APIs or even ignore them...
That’s what we named **Da Real Fragmentation** and it makes development much more difficult. In addition (as shown in the series), if you have background processes running, your life becomes a nightmare!

As you may have noticed, finally I didn't mention any of the problems that we found creating [Drivies](https://drivies.onelink.me/692963081?pid=ownmedia&c=pguardiola) nor how we have solved such fragmentation issues. We'll do that in the [talk](http://2016.codemotion.es/agenda.html#5732408326356992/84644004)! So come, join us and do not let them tell you!

![Hype](https://raw.githubusercontent.com/Guardiola31337/guardiola31337.github.io/master/art/darealfragmentation/sensors/hype.gif)

Don't worry if you can't attend! I'll post the slides and the video later ;)

___

P.S I hope you enjoyed the series. It was a pleasure to share with you our discoveries. I encourage you to watch the video of the talk when available. As I said, I'll let you know!
BTW, this doesn’t have to end here, I’m really looking forward to hearing from you so we don’t stop learning. Please, feel free to leave a comment below and/or send a Pull Request on [DaRealFragmentation](https://github.com/Guardiola31337/darealfragmentation) repository!