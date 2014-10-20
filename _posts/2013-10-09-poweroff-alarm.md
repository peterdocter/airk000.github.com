---
layout: post
title: "Android关机闹钟（高通平台）"
description: ""
category: "android"
tags: [android]
---

Android关机闹钟在高通平台实现方案
===

<br />
###内核驱动接口

#####drivers/rtc/Kconfig

    @@ -111,6 +111,12 @@ config RTC_INTF_ALARM_DEV
       help
         Exports the alarm interface to user-space.
     
    +config NEW_FEATURE_POWEROFF_ALARM
    +  bool "Android power off alarm"
    +  default y
    +  help
    +    Make the mobile phone can be power-up by alarm
    +
     config ANDROID_RTC_CHANGE_WAIT
       bool "Android rtc wait event"
       depends on RTC_INTF_ALARM_DEV 

#####drivers/rtc/alarm-dev.c

    @@ -25,6 +25,8 @@
     #include <linux/uaccess.h>
     #include <linux/wakelock.h>
     
    +#ifdef CONFIG_NEW_FEATURE_POWEROFF_ALARM
    +#include <mach/pmic.h>
    +#endif
    +
     #define ANDROID_ALARM_PRINT_INFO (1U << 0)
     #define ANDROID_ALARM_PRINT_IO (1U << 1)
     #define ANDROID_ALARM_PRINT_INT (1U << 2)
    @@ -182,6 +184,35 @@ static long alarm_ioctl(struct file *file, unsigned int cmd, unsigned long arg)
         if (rv < 0)
           goto err1;
         break;
    +#ifdef CONFIG_NEW_FEATURE_POWEROFF_ALARM
    +    case ANDROID_ALARM_SET_POWERUP_RTC:
    +    {
    +      #pragma message("POWERUP ALARM has been compiled.")
    +        uint32_t rtc_alarm_time;
    +        struct rtc_time rtc_now;
    +        if (copy_from_user(&rtc_alarm_time, (void __user *)arg,
    +                    sizeof(rtc_alarm_time))) {
    +            rv = -EFAULT;
    +            goto err1;
    +        }
    +        printk("\n rtc_alarm_time = %d\n", rtc_alarm_time);
    +
    +        if (rtc_alarm_time > 0) {
    +            if (pmic_rtc_get_time(&rtc_now) < 0) {
    +                rtc_now.sec = 0;
    +                if (pmic_rtc_start(&rtc_now) < 0) {
    +                    printk("Get and set rtc info failed!\n");
    +                    break;
    +                }
    +            }
    +            pmic_rtc_disable_alarm(PM_RTC_ALARM_1);
    +            rtc_now.sec += rtc_alarm_time;
    +            pmic_rtc_enable_alarm(PM_RTC_ALARM_1, &rtc_now);
    +            printk("\n rtc_now: sec = %d\n", rtc_now.sec);
    +        }
    +        break;
    +    }
    +#endif
       case ANDROID_ALARM_GET_TIME(0):
         switch (alarm_type) {
         case ANDROID_ALARM_RTC_WAKEUP: 

#####include/linux/android_alarm.h

    @@ -105,6 +105,9 @@ enum android_alarm_return_flags {
     #define ANDROID_ALARM_SET_AND_WAIT(type)    ALARM_IOW(3, type, struct timespec)
     #define ANDROID_ALARM_GET_TIME(type)        ALARM_IOW(4, type, struct timespec)
     #define ANDROID_ALARM_SET_RTC               _IOW('a', 5, struct timespec)
    +#ifdef CONFIG_NEW_FEATURE_POWEROFF_ALARM
    +#define ANDROID_ALARM_SET_POWERUP_RTC       _IOW('a', 7, struct timespec)
    +#endif
     #define ANDROID_ALARM_BASE_CMD(cmd)         (cmd & ~(_IOC(0, 0, 0xf0, 0)))
     #define ANDROID_ALARM_IOCTL_TO_TYPE(cmd)    (_IOC_NR(cmd) >> 4)
  

添加完后在相应设备的defconfig中添加CONFIG_NEW_FEATURE_POWEROFF_ALARM=y即可。

###上层API接口

#####frameworks/base/core/java/android/app/AlarmManager.java

     +/* 
     + * New API for power off alarm
     + * @hide
     + */
     +public void setPowerUpalarm(long triggerAtTime) {
     +    try {
     +        mService.setPowerUpalarm(triggerAtTime);
     +    } catch (RemoteException ex) {
     +    }
     +}

#####frameworks/base/core/java/android/app/IAlarmManager.aidl


    interface IAlarmManager {
         void set(int type, long triggerAtTime, in PendingIntent operation);
         void setRepeating(int type, long triggerAtTime, long interval, in PendingIntent operation);
         void setInexactRepeating(int type, long triggerAtTime, long interval, in PendingIntent operation);
         void setTime(long millis);
         void setTimeZone(String zone);
         void remove(in PendingIntent operation);
    +    void setPowerUpalarm(long triggerAtTime);
    }

#####frameworks/base/services/java/com/android/server/AlarmManagerService.java


    +/*
    + *@hide
    + */
    +public void setPowerUpalarm(long triggerAtTime)
    +{
    +    if(triggerAtTime == 0)
    +        return;
    +    synchronized (mLock) {
    +        Alarm alarm = new Alarm();
    +        alarm.when = triggerAtTime;
    +        alarm.repeatInterval = 0;
    +
    +        if (localLOGV) Slog.v(TAG, "set: " + alarm);
    +        if (mDescriptor != -1){
    +        // The kernel never triggers alarms with negative wakeup times
    +        // so we ensure they are positive.
    +        long alarmSeconds, alarmNanoseconds;
    +
    +        if (alarm.when < 0) {
    +            alarmSeconds = 0;
    +            alarmNanoseconds = 0;
    +        } else {
    +            alarmSeconds = alarm.when / 1000;
    +            alarmNanoseconds = (alarm.when % 1000) * 1000 * 1000;
    +        }
    +            setPowerUpalarm(mDescriptor, alarmSeconds, alarmNanoseconds);
    +        } else{
    +            Message msg = Message.obtain();
    +            msg.what = ALARM_EVENT;
    +
    +            mHandler.removeMessages(ALARM_EVENT);
    +            mHandler.sendMessageAtTime(msg, alarm.when);
    +        }
    +    }
    +}
    
     private native int waitForAlarm(int fd);
     private native int setKernelTimezone(int fd, int minuteswest);
    +private native void setPowerUpalarm(int fd, long seconds, long nanoseconds);

#####frameworks/base/services/jni/com_android_server_AlarmManagerService.cpp


    + static void android_server_AlarmManagerService_setPowerUpalarm(JNIEnv* env, jobject obj, jint fd, jlong seconds, jlong nanoseconds)
    + {
    +     struct timespec ts; 
    +     ts.tv_sec = seconds;
    +     ts.tv_nsec = nanoseconds;
    + 
    +     int result = ioctl(fd, ANDROID_ALARM_SET_POWERUP_RTC, &ts);
    +     if (result < 0)
    +             {   
    +             ALOGE("Unable to set power off alarm to %lld.%09lld: %s\n", seconds, nanoseconds, strerror(errno));
    +             }   
    + }

     static JNINativeMethod sMethods[] = { 
          /* name, signature, funcPtr */
         {"init", "()I", (void*)android_server_AlarmManagerService_init},
         {"close", "(I)V", (void*)android_server_AlarmManagerService_close},
         {"set", "(IIJJ)V", (void*)android_server_AlarmManagerService_set},
         {"waitForAlarm", "(I)I", (void*)android_server_AlarmManagerService_waitForAlarm},
         {"setKernelTimezone", "(II)I", (void*)android_server_AlarmManagerService_setKernelTimezone},
    +    {"setPowerUpalarm", "(IJJ)V", (void*)android_server_AlarmManagerService_setPowerUpalarm},
     };

###静默实现

以下代码实现的效果是每次关机的时候检测当前是否存在需要之后相应的闹钟，如果有则调用此接口。

#####frameworks/base/services/java/com/android/server/pm/ShutdownThread.java (<= Android4.1)
#####or
#####frameworks/base/services/java/com/android/server/power/ShutdownThread.java (>= Android4.2)


    + import android.util.shendu.ShenduAlarm;
    + import android.app.AlarmManager;
    
    + private static final int ALARM_BEFORE_TIME = 90*1000;   // 1 and a half minutes, 90,000 milesecs
    
    private static void beginShutdownSequence(Context context) {
        synchronized (sIsStartedGuard) {
            if (sIsStarted) {
                Log.d(TAG, "Shutdown sequence already running, returning.");
                return;
            }
            sIsStarted = true;
        }
        ...
        ...
        sInstance.mContext = context;
        sInstance.mPowerManager = (PowerManager)context.getSystemService(Context.POWER_SERVICE);
        + AlarmManager am =(AlarmManager)context.getSystemService(Context.ALARM_SERVICE);
        + long alarmTime = ShenduAlarm.getClockTime(context);
        + am.setPowerUpalarm(alarmTime-System.currentTimeMillis()-ALARM_BEFORE_TIME);
    
        // make sure we never fall asleep again
        sInstance.mCpuWakeLock = null;

关于ShenduAlarm这个可以参考<https://github.com/ShenduOS/android_frameworks_base/blob/jellybean/core/java/android/util/shendu/ShenduAlarm.java>，目的是获取当前手机所设置的闹钟。

<br />
这样关机闹钟就添加完毕了，最终效果是只要设定了闹钟，无论开关机都会开机去响（假设设备一直有电）。

<br />
>PS: 可能存在的问题是有的设备可能会一直记得之前设定过的值所以造成第一次设定后，以后不停的关机就自动重启，解决办法是在内核驱动中当判断上层传下的参数为0时，给出一个比较大的数值，比如604800秒即420天。
