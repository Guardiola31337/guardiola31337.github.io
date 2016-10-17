---
layout: post
title: Da Real Fragmentation - Alarms
summary: In this first article, we are going to analyze alarms in Android.
---

In order to prepare the [talk](http://2016.codemotion.es/agenda.html#5732408326356992/84644002), that my friend [Raúl](https://twitter.com/rromanl) and I will give at [Codemotion Spain 2016](http://2016.codemotion.es/), I decided to write a series of posts explaining how **alarms** and **sensors** work and how **Doze** mode affects their normal behavior. In the talk, we'll discuss those topics, the problems that we found along the way creating [Drivies](http://www.driviesapp.com/) and how we have solved such fragmentation issues. Don't miss it!

In this first article, we are going to analyze alarms in Android. Let's go!

<!-- more -->

![MinionAlarm](https://raw.githubusercontent.com/Guardiola31337/guardiola31337.github.io/master/art/darealfragmentation/alarms/minion_alarm.gif)

## Alarms

Eventually you might need to schedule a task in background. There are a lot of scenarios when you need to execute something out of the public eye, even if your app is not running. For example, you might need to send a notification at some later time or to check periodically if an event occurred. To implement that kind of operation we use _alarms_.

Scheduled work on Android can be implemented differently according to _frequency_, _foreground vs. background_ and your `minSdkVersion`. In some cases, you can use a simple `postDelayed()` loop. But, as you may already know, it only works if your process is still alive. If you no longer have a process, the better way to handle the job is to use `AlarmManager`. `JobScheduler`, introduced in [Lollipop](https://developer.android.com/about/versions/lollipop.html), does the same. For the sake of simplicity, we are going to focus firstly on `AlarmManager`. We'll explore the different configuration options, explaining their pros and cons showing how [Doze](https://developer.android.com/training/monitoring-device-state/doze-standby.html) mode affects each. `JobScheduler` will be covered in a future post.

### Options

In order to set up _alarms_, first of all, we have to know the different options available. Let's review them in depth.

* We should analyze if our alarms have to **wake up** the device or not. As you can imagine, devices enter into _Sleep mode_ after a certain period of inactivity. This means that the CPU of the device has nothing to do and the system stops it. In this state, you can only count on GSM or CDMA radio and `AlarmManager`. Anything keeping a `WakeLock` maintains the device awake, thereby preventing the system from entering this mode. So, if you need to ensure that the alarm will be fired at the scheduled time, you should use the _wakeup_ version. But be careful, because this is not recommendable in the interest of battery life. Alternatively, if you don't use the wakeup style and the device is asleep, alarms will fire when the device is next awake.

* We are able to configure the **rate** scheduling, as well. In fact, we can either create _one-shot_ alarms, or _repeating_ alarms at a regular period of your preference. Beware of repeating alarms because if you don't design them properly, you could end up abusing system resources.

* Another decision to take into account is how **precise** your alarms need to be. There are two options: _exact_ and _inexact_. _Exact_ alarms will fire at the scheduled time whereas, in the _inexact_ ones, Android synchronizes multiple inexact alarms around the same time somewhat, reducing the drain on the battery. Logically, you should avoid using exact alarms if possible.

* Android supports two **clock** types for alarm service: _elapsed real time_ and _real time clock_ (RTC). The first corresponds to the time since the device last booted (a time relative to now) and the second uses UTC time (an absolute time). This means that elapsed real time is suited to set alarms that should fire based on the passage of time (most of the polling operations). If you want to set an alarm which is dependent on _real world time_ (current locale), then the real time clock is more suitable (alarm clocks or calendar alerts). Note that elapsed real time is the better choice because if the user changes the device's locale or their time settings, it could cause some unexpected behavior in the app. Even more, using a real time clock alarm clock does not scale well because if you set a concrete time for an app to sync a server, the load on the server could result in high latency or be overwhelmed when all instances hit at the same time.

* Finally, we have to tell Android **what to do** when alarms occur. We will do that supplying a `PendingIntent`.

### Types

Below is the list of the different alarm types available.

* [ELAPSED_REALTIME](https://developer.android.com/reference/android/app/AlarmManager.html#ELAPSED_REALTIME): Fires the pending intent based on the amount of time since the device was booted, but doesn't wake up the device. The elapsed time includes any time during which the device was asleep.

* [ELAPSED_REALTIME_WAKEUP](https://developer.android.com/reference/android/app/AlarmManager.html#ELAPSED_REALTIME_WAKEUP): Wakes up the device and fires the pending intent after the specified length of time has elapsed since device boot.

* [RTC](https://developer.android.com/reference/android/app/AlarmManager.html#RTC): Fires the pending intent at the specified time but does not wake up the device.

* [RTC_WAKEUP](https://developer.android.com/reference/android/app/AlarmManager.html#RTC_WAKEUP): Wakes up the device to fire the pending intent at the specified time.

### Methods

There are a bunch of methods that you can call on `AlarmManager` to establish an alarm. Following we are going to describe each.

* [`set()`](https://developer.android.com/reference/android/app/AlarmManager.html#set(int, long, android.app.PendingIntent)): Schedule an alarm. Added in API level 1. On Android 4.3 (API level 18) and below `set()` has the same behavior as `setExact()`. However, beginning in API 19, the trigger time passed to this method is treated as inexact and alarms can be deferred and delivered any time thereafter. Like other _inexact_ policies, the goal is to batch alarms together in order to minimize the number of wakeups and improve battery efficiency. Note that alarms' actual delivery ordering may not match the order of their requested delivery times. If your application has strong ordering requirements use `setExact()` or `setWindow()`.

* [`setExact()`](https://developer.android.com/reference/android/app/AlarmManager.html#setExact(int, long, android.app.PendingIntent)): Schedule an alarm to be delivered precisely at the stated time. Added in API level 19. On Android 4.4 and higher, `setExact()` is used for one-shot alarms.

* [`setWindow()`](https://developer.android.com/reference/android/app/AlarmManager.html#setWindow(int, long, long, android.app.PendingIntent)): Schedule an alarm to be delivered within a given window of time. Added in API level 19. It allows the application to precisely control the degree to which its delivery might be adjusted by the OS.

* [`setRepeating()`](https://developer.android.com/reference/android/app/AlarmManager.html#setRepeating(int, long, long, android.app.PendingIntent)): Schedule a repeating alarm. Added in API level 1. It is used for alarms that should be fired at specific points in time at a specific frequency. A repeating alarm has the following characteristics:
  * An alarm type. Specified by the units of time to use and whether or not it should occur when the device is in sleep mode. See [Types](#types) explained above.

  * A trigger time. Time in milliseconds when we want the first event to fire. If the trigger time you specify is in the past, the alarm triggers immediately.

  * The alarm's interval. Interval in milliseconds between subsequent repeats of the alarm. However, note that as of Android 5.1, your minimum interval is one minute (60000ms); values less than that will be rounded up to one minute. This minimum interval is enforced regardless of your `targetSdkVersion` value.

  * The `PendingIntent` to invoke when the alarm is triggered. When you set a second alarm that uses the same pending intent, it replaces the original alarm.

  **Warning**: As of Android 4.4 and above, `setRepeating()` behaves identically to `setInexactRepeating()`, which means, all repeating alarms are inexact. If your application needs precise delivery times then it must use one-time alarms, scheduling the next one yourself when handling each alarm delivery. Ideally, you should use `setInexactRepeating()` for the reasons already mentioned.

* [`setInexactRepeating()`](https://developer.android.com/reference/android/app/AlarmManager.html#setInexactRepeating(int, long, long, android.app.PendingIntent)): Schedule a repeating alarm that has inexact trigger time requirements. Added in API level 3. Android can adjust alarms' delivery times to cause them to fire simultaneously, avoiding waking the device from sleep more than necessary. Your alarm's first trigger will not be before the requested time, but it might not occur for almost a full interval after that time. Prior to API 19, if the general frequency is one of `INTERVAL_FIFTEEN_MINUTES`, `INTERVAL_HALF_HOUR`, `INTERVAL_HOUR`, `INTERVAL_HALF_DAY` or `INTERVAL_DAY` then the alarm will be phase-aligned with other alarms to reduce the number of wakeups. Otherwise, the alarm will be set as though the application had called `setExactRepeating()`. For apps whose `targetSdkVersion` is set to 19 or higher, all repeating alarms will be inexact and subject to batching with other alarms regardless of their stated repeat interval.

* [`setAlarmClock()`](https://developer.android.com/reference/android/app/AlarmManager.html#setAlarmClock(android.app.AlarmManager.AlarmClockInfo, android.app.PendingIntent)): Schedule an alarm that represents an alarm clock. Added in API level 21. The system may choose to display information about this alarm to the user. This method is like `setExact()`, but implies `RTC_WAKEUP`.

* [`setAndAllowWhileIdle()`](https://developer.android.com/reference/android/app/AlarmManager.html#setAndAllowWhileIdle(int, long, android.app.PendingIntent)): Like `set()`, but this alarm will be allowed to execute even when the system is in low-power idle modes. Added in API level 23. When the alarm is dispatched, the app will also be added to the system's temporary whitelist for approximately 10 seconds to allow that application to acquire further wake locks in which to complete its work. To reduce abuse, there are restrictions on how frequently these alarms will go off for a particular application. Under normal system operation, it will not dispatch these alarms more than about every minute; when in low-power idle modes this duration may be significantly longer, such as 15 minutes. Unlike other alarms, the system is free to reschedule this type of alarm to happen out of order with any other alarms, even those from the same app. Regardless of the app's target SDK version, this call always allows batching of the alarm.

* [`setExactAndAllowWhileIdle()`](https://developer.android.com/reference/android/app/AlarmManager.html#setExactAndAllowWhileIdle(int, long, android.app.PendingIntent)): Like `setExact()`, but this alarm will be allowed to execute even when the system is in low-power idle modes. Added in API level 23. If you don't need exact scheduling of the alarm but still need to execute while idle, consider using `setAndAllowWhileIdle()`. This method has the same characteristics and restrictions as `setAndAllowWhileIdle()`. Note that the OS will allow itself more flexibility for scheduling these alarms than regular exact alarms, since the application has opted into this behavior. When the device is idle it may take even more liberties with scheduling in order to optimize for battery life.

**Warning**: On Android 5.1 and higher, alarms must be set to occur at least 5 seconds in the future from now. You cannot trigger an alarm to occur in the future sooner than 5 seconds.

### Conclusion

Can you spot the fragmentation issues??

See the following flowchart...

![Pro-tipAlarmsFlowchartv2](https://raw.githubusercontent.com/Guardiola31337/guardiola31337.github.io/master/art/darealfragmentation/alarms/Pro-tipAlarmsFlowchartv2.png)

First time I saw it I was all like...

![AlanThinkingHard](https://raw.githubusercontent.com/Guardiola31337/guardiola31337.github.io/master/art/darealfragmentation/alarms/alan_thinking_hard.gif)

As you may have noticed, working with alarms in Android entails dealing with SDK fragmentation. There a bunch of APIs which do the same thing and you have to master them in advance to ensure your scheduled tasks will work properly. The names of the methods are almost identical and you have to be careful choosing the one that best suits your needs depending on the API level you are targeting. Even some of them are lying based on which OS version is running! That's really confusing and annoying. Don't you think so? And I didn't mention what manufacturers do with them... Yet!

Stay tuned!

___

In the next post, we'll talk about how _Doze_ and _App Standby_ work and how these power-savings optimizations influence alarms.
In the meantime, you can check our [playground](https://github.com/Guardiola31337/darealfragmentation) sample repository on Github.

P.S. Quick reminder! Feel free to leave a comment below. I'm looking forward to hearing from you.