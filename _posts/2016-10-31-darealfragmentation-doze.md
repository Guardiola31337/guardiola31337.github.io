---
layout: post
title: Da Real Fragmentation - Doze
summary: In this post, we'll dive into Doze and App Standby modes, explaining how these two power-saving features work in detail. Furthermore, we’ll go deeper, describing how alarms are affected when devices enter Doze mode and the problems it causes on apps with a strong background component.
---

In the last [post](http://pguardiola.com/blog/darealfragmentation-alarms/), we learnt how Android _Alarms_ work. We analyzed the different options, types and methods available and realized that scheduling tasks in Android implies fragmentation issues. In this post, we'll dive into [Doze and App Standby](https://developer.android.com/training/monitoring-device-state/doze-standby.html) modes, explaining how these two power-saving features work in detail. Furthermore, we’ll go deeper, describing how alarms are affected when devices enter _Doze_ mode and the problems it causes on apps with a strong background component. Stay awake!

<!-- more -->

![Shaq-dozing-off](https://raw.githubusercontent.com/Guardiola31337/guardiola31337.github.io/master/art/darealfragmentation/doze/Shaq-dozing-off.gif)

## Doze

Starting from Android 6.0 (API level 23), Android introduced a new way for devices to preserve battery life by going idle. _Doze_ reduces battery consumption by deferring background CPU and network activity for apps when the device is unused for long periods of time. While in _Doze_, your scheduled alarms (with `AlarmManager`), jobs (with `JobScheduler`), and syncs (with `SyncManager`) will be ignored by default, except during occasional _idle maintenance windows_. Periodically, the system exits _Doze_ for a brief time to let apps complete their deferred activities. During these _maintenance windows_, the system runs all pending syncs, jobs, and alarms, and lets apps access the network. Note that activating the screen or plugging in the device exits _Doze_ and removes these processing restrictions.

### Types

First of all, we have to know the different _stages_ of _Doze_ mode and when they will be active on devices. Let’s review them in depth.

#### Deep-Doze

_Deep-Doze_ was introduced with Android Marshmallow and it's activated when the **device** is **unplugged**, when the **screen** is **off**, and when **no motion** has been **detected** for some time. At this point, the OS tags internally when these conditions were met, waits about 30 minutes to be sure, and then jumps into the _Deep-Doze_ mode. As I already mentioned, _Deep-Doze_ provides short maintenance windows where apps can sync up their latest data. At the conclusion of each maintenance window, the system again enters Doze, suspending network access and deferring jobs, syncs, and alarms. Over time, the system schedules maintenance windows less and less frequently, helping to reduce battery consumption in cases of longer-term inactivity when the device is not connected to a charger. 

A picture is worth a thousand words!

![Deep-Doze](https://raw.githubusercontent.com/Guardiola31337/guardiola31337.github.io/master/art/darealfragmentation/doze/deep-doze.png)

Maybe not this time...

#### Light-Doze

Android 7.0 introduced _Light-Doze_, a more lightweight version of _Doze_. This mode brings a subset of CPU and network restrictions while the **device** is **unplugged** with the **screen** turned **off**, but **not** necessarily **stationary**. In this case, Android checks these conditions during a couple minutes before applying the first subset of restrictions: It shuts off app network access, and defers jobs and syncs. If you depend on network access and you are using `AlarmManager` for your recurring background work, consider using `JobScheduler` so that you only start up when the network is available in the maintenance windows. As you can see in the following diagram, the period between data-syncing maintenance windows is shorter than in _Deep-Doze_.

![Light-Doze](https://raw.githubusercontent.com/Guardiola31337/guardiola31337.github.io/master/art/darealfragmentation/doze/light-doze.png)

If the device is stationary for a certain time after entering _Light-Doze_, the system applies the rest of the _Deep-Doze_ restrictions to `WakeLock`, `AlarmManager`, GPS, and Wi-Fi scans. Regardless of whether some or all _Doze_ restrictions are being applied, again, the system wakes the device for brief maintenance windows, during which applications are allowed network access and can execute any deferred jobs/syncs.

![Light-Doze-To-Deep-Doze](https://raw.githubusercontent.com/Guardiola31337/guardiola31337.github.io/master/art/darealfragmentation/doze/light-doze-to-deep-doze.png)

It is also important to note that _Deep-Doze_ can be interrupted and stepped up to _Light-Doze_ if the device stops being stationary but still not charging with the screen off.

## App Standby

_App Standby_ lets the system determine that an app which has not been in the foreground (or is showing a `Notification`), after some undefined period of time not being used by the user, is idle. While the app is in _Standby_, and the device unplugged, the app behaves similarly to _Doze_ mode, which means that the OS doesn't permit it to access the network and to execute jobs and syncs. If the device is idle for long periods of time, Android only allows idle apps network access around once a day. In the moment the user plugs the device into a power supply, everything returns to normal.

**Warning**: _Doze_ and _App Standby_ manage the behavior of all apps running on Android 6.0 or higher, regardless of whether they are specifically targeting API level 23.

## What restrictions does Doze add?

Now that we know how _Doze_ works, we are going to analyze the limitations that it introduces and how to wake the device from it, so that your scheduled and background work is not affected.

Some argue that _Doze_ shouldn't disrupt your app flow too much, because in _Light-Doze_ maintenance windows occur frequently and _Deep-Doze_ only affects your app when the user is not using their device at all. Of course, in favor of battery life...

![Agreed-But-Not-Really](https://raw.githubusercontent.com/Guardiola31337/guardiola31337.github.io/master/art/darealfragmentation/doze/agreed-but-not-really.gif)

But, what about real-time apps or time-sensitive apps that do their job behind the scenes without user interaction? If you are working on any of these kind of apps, you definitely need a way to get around _Doze_'s effects. Let's check the options available.

### Alarms

As you may already know, _Doze_ is particularly likely to affect activities that `AlarmManager` alarms and timers manage. Be careful, because if you use the alarms methods from Android 5.1 (API level 22) or lower when the system is in _Doze_ (Android 6.0+ devices), alarms won't fire.

As explained in the previous [post](http://pguardiola.com/blog/darealfragmentation-alarms/), Android 6.0 (API level 23) introduced two new `AlarmManager` methods: `setAndAllowWhileIdle()` and `setExactAndAllowWhileIdle()`. With these methods, you can set alarms that will fire even if the device is in Doze and not yet in a maintenance window. Remember that you only have 10 seconds to capture further wake locks in which to complete your work.

**Warning**: Neither `setAndAllowWhileIdle()` nor `setExactAndAllowWhileIdle()` can fire alarms more than once per 9 minutes, per app.

From the point of view of power management, _Doze_ mode leaves `setAlarmClock()` events alone. Using this method, _Doze_ exits completely shortly before the alarm goes off regardless of device state, so the app is able to fetch fresh data before the user is back. The major problem with `setAlarmClock()` is that it is visible to the user, because it represents an alarm clock (as its name suggests). Which implies that the user will see the alarm clock icon in the status bar and the scheduled time at which the phone will alert the user if the notification shade is fully opened.

### Firebase Cloud Messaging

If your app requires a persistent connection to the network to receive critical messages, you could use [Firebase Cloud Messaging](https://firebase.google.com/docs/cloud-messaging/) (formerly [GCM](https://developers.google.com/cloud-messaging/)). FCM **high-priority** messages let you reliably wake your app to access the network, even if the user’s device is in _Doze_ or the app is in _App Standby_ mode. The system delivers the message and gives the app temporary access to network services and partial wakelocks, then returns the device or app to idle state. As you can imagine, this option introduces some difficulties, e.g. you have to setup FCM, the app needs network connectivity, etc.

### Whitelist

The system provides a configurable whitelist of apps that are partially exempt from _Doze_ and _App Standby_ optimizations. This _whitelist_ of apps allows you to hold wakelocks and access the network. However, **other restrictions still apply** to the _whitelisted_ app, just as they do to other apps. It does not change the behavior of `AlarmManager`, `JobScheduler` or `SyncManager`. You can call [`isIgnoringBatteryOptimizations()`](https://developer.android.com/reference/android/os/PowerManager.html#isIgnoringBatteryOptimizations(java.lang.String)) to check if your app is already on the _whitelist_. As described in the [documentation](https://developer.android.com/training/monitoring-device-state/doze-standby.html#support_for_other_use_cases), users can manually configure the _whitelist_ in Settings > Battery > Battery Optimization and, alternatively, the system provides ways for apps to ask users to whitelist them.

* An app can fire the [`ACTION_IGNORE_BATTERY_OPTIMIZATION_SETTINGS`](https://developer.android.com/reference/android/provider/Settings.html#ACTION_IGNORE_BATTERY_OPTIMIZATION_SETTINGS) intent to take the user directly to the Battery Optimization, where they can add the app.

* An app holding the [`REQUEST_IGNORE_BATTERY_OPTIMIZATIONS`](https://developer.android.com/reference/android/Manifest.permission.html#REQUEST_IGNORE_BATTERY_OPTIMIZATIONS) permission can trigger a system dialog to let the user add the app to the _whitelist_ directly, without going to settings. The app fires a [`ACTION_REQUEST_IGNORE_BATTERY_OPTIMIZATIONS`](https://developer.android.com/reference/android/provider/Settings.html#ACTION_REQUEST_IGNORE_BATTERY_OPTIMIZATIONS) intent to trigger the dialog.

**Warning**: You have to be careful because Google Play policies prohibit apps from requesting direct exemption from Power Management features in Android 6.0+ (_Doze_ and _App Standby_) unless the core function of the app is adversely affected. So, if in doubt, make sure your app is in the [acceptable use cases](https://developer.android.com/training/monitoring-device-state/doze-standby.html#whitelisting-cases) for _whitelisting_.

### Foreground Service

Although is not officially documented, apps that have been running foreground services (with the associated notification) are not restricted by _Doze_.

### Hope

_Doze_ is for the entire device, so one last case is hoping somebody else interrupts it. You may be thinking...

![Hope-Is-Dangerous](https://raw.githubusercontent.com/Guardiola31337/guardiola31337.github.io/master/art/darealfragmentation/doze/hope-is-dangerous.gif)

Obviously, it's unlikely, but it's still an option :)

## Conclusion

We've learnt how to live with _Doze_ and _App Standby_ modes and a bunch of workarounds to avoid the issues caused by them. Apart from that and in my opinion, a more philosophical problem has been clearly shown: Android is adding more and more restrictions, especially regarding background processes. Like [ours](http://www.driviesapp.com/), lots of other apps are being severely compromised by these changes. I'm starting to feel that we are loosing that _openness_ that we used to have... We'll see how it ends...

Anyway, say Boo! today and...

![Pumpkin-Dance](https://raw.githubusercontent.com/Guardiola31337/guardiola31337.github.io/master/art/darealfragmentation/doze/pumpkin-dance.gif)

___

In the [next](http://pguardiola.com/blog/darealfragmentation-sensors/) and final article, we'll talk about _sensors_. We'll explain how they work. Besides, we'll introduce some peculiarities when you are using them and battery optimization modes show up and show how to deal with such problems.

P.S. I hope you’re enjoying the series but I would love to hear your thoughts! As always, feel free to leave a comment below and/or send a Pull Request on [DaRealFragmentation](https://github.com/Guardiola31337/darealfragmentation) repository!