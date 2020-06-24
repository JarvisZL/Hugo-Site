# Android Monkey RunMonkeyCycles



## run中调用runMonkeyCycles
- 在run函数中获取完对应的EventSource并validate之后，就会调用runMonkeyCycle函数开始执行event
```kotlin
mNetworkMonitor.start()
var crashedAtCycle : Int
try {
    /** where the magic happens*/
    crashedAtCycle = runMonkeyCycles()
} finally {
    //Release the rotation lock if it's still held and restore the original orientation
    MonkeyRotationEvent(Surface.ROTATION_0, false).injectEvent(mWm!!, mAm!!, mVerbose)
}
mNetworkMonitor.stop()
```

## runMonkeyCycles
- 在进行一系列初始化之后进入循环
```kotlin
while (!systemCrashed) {
    //如果设置了运行多久，则使用time判断，否则使用count判断
    if (mRunningMillis < 0) {
        if (cycleCounter >= mCount) break
    } else {
        currentTime = SystemClock.elapsedRealtime()
        if (currentTime > mEndTime) break
    }
    //一系列信息report
    ....

    //get the next event
    val ev = mEventSource.getNextEvent()
    if (ev != null) {
        var failed = false
        when(ev.injectEvent(mWm!!, mAm!!, mVerbose)) {
            MonkeyEvent.INJECT_FAIL -> {
                for (i in 0..4){
                    if (mVerbose >= 1) {
                        Logger.iLPrintln("Injection Faild, retry No. $i")
                    }
                    Thread.sleep(50)
                    if (ev.injectEvent(mWm!!,mAm!!,mVerbose) != MonkeyEvent.INJECT_FAIL)
                        break
                }
                Logger.lPrintln("   //Injection Failed ${ev.eventType} ${ev.eventId}, Given up")
                failed = true
                when (ev) {
                    is MonkeyKeyEvent -> mDroppedKeyEvents++
                    is MonkeyMotionEvent -> mDroppedPointerEvents++
                    is MonkeyFlipEvent -> mDroppedFlipEvents++
                    is MonkeyRotationEvent -> mDroppedRotationEvents++
                }
            }
            MonkeyEvent.INJECT_ERROR_REMOTE_EXCEPTION -> {
                systemCrashed = true
                System.err.println("** Error: RemoteException while injecting event.")
            }
            MonkeyEvent.INJECT_ERROR_SECURITY_EXCEPTION -> {
                systemCrashed = !mIgnoreSecurityExceptions
            }
        }

        // Don't count throttling as an event
        if (ev !is MonkeyThrottleEvent && !failed) {
            eventCounter++
            if (mCountEvents){
                cycleCounter++
            }
        }
    } else {
        if (!mCountEvents) {
            cycleCounter++
            writeScriptLog(cycleCounter)
            // Capture the bugreport after n iteration
            if (mGetPeriodicBugreport) {
                if (cycleCounter % mBugreportFrequency == 0L) {
                    mRequestPeriodicBugreport = true
                }
            }
        } else {
            // Event Source has signaled that we have no more events to
            // process
            break
        }
    }    
}
```
- 通过mEventSource.getNextEvent()来获得下一个执行的Event。
- 然后通过ev.injectEvent(...)注入事件。

```kotlin
//返回注入的事件数量
return eventCounter
```
