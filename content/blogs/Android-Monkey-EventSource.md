---
author: "Lingyu Zhang"
author_link: "https://jarviszly.com"
title: "Android Monkey EventSource"
date: 2020-06-17T17:33:09+08:00
lastmod: 2020-06-17T17:33:09+08:00
draft: false
description: ""
show_in_homepage: true
description_as_summary: false
license: ""

tags: ["Android","Monkey"]
categories: []

featured_image: ""
featured_image_preview: ""

comment: true
toc: true
auto_collapse_toc: true
math: true
---

## run函数中申明不同的EventSource
- **脚本运行**：命令中使用了-f且只有一个脚本则直接使用该脚本进行测试
```kotlin
mScriptFileNames.size == 1 -> {
    //script mode, ignore other options
    mEventSource = MonkeySourceScript(mRandom!!, mScriptFileNames[0]!!, 
        mThrottle, mRandomizeThrottle, mProfileWaitTime, mDeviceSleepTime)
    mEventSource.setVerbose(mVerbose)
    mCountEvents = false
}
```
- **随机脚本运行**：命令中使用了-f且包含多个脚本则随机运行这些脚本进行测试
- 如果有参数setup,则先运行带-setup的脚本用于配置环境等
```kotlin  
mScriptFileNames.size > 1 -> {
    mEventSource = if (mSetupFileName != null) {
            mCount++
            MonkeySourceRandomScript(mSetupFileName, mScriptFileNames, mThrottle,
                mRandomizeThrottle, mRandom!!, mProfileWaitTime, mDeviceSleepTime, 
                mRandomizeScript)
        } else {
            MonkeySourceRandomScript(mScriptFileNames, mThrottle, mRandomizeScript, 
                mRandom!!,mProfileWaitTime, mDeviceSleepTime, mRandomizeScript)
        }
    mEventSource.setVerbose(mVerbose)
    mCountEvents = false
}
```
- **远程运行**：如果monkey参数包含“–port”，则通过MonkeySourceNetwork()运行远程调用
```kotlin
 //TODO: implement network-source Monkey
mServerPort != -1 -> {
    throw Exception("not implemented yet!")
}
```
- **随机执行(default)**

```kotlin
when{
    ...
    else -> {
        // random source by default
        //check seeding performance
        if (mVerbose >= 2) { Logger.lPrintln("// Seeded: $mSeed") }

        mEventSource = MonkeySourceRandom(mRandom!!, mMainApps, mThrottle, mRandomizeThrottle,
            mPermissionTargetSystem)
        mEventSource.setVerbose(mVerbose)

        //set any of the factors that has been set
        mFactors.forEachIndexed { index, fl ->
            if (fl <= 0f) {
                (mEventSource as MonkeySourceRandom).setFactors(index, fl)
            }
        }

        //in random mode, we start with a random activity
        (mEventSource as MonkeySourceRandom).generateActivity()
    }
}
```

- 获取对应的EventSource后，会调用相应的validate()函数来验证并调整各个event所占的比例。
```kotlin
// validate source generator
if (!mEventSource.validate()) { return -5 }
```

## MonkeySourceRandom分析
- 该文件对应了默认的事件生成方式。
- 当在run中声明这个类的一个对象的时候会调用内部的init, 给各个事件初始化一个比例。
```kotlin
init {
    // default values for random distributions
    // note, these are straight percentages, tomatch user input (cmd line args)
    // but they will be converted to 0..1 valuesbefore the main loop runs.
    mFactors[FACTOR_TOUCH] = 25.0f
    mFactors[FACTOR_MOTION] = 20.0f
    mFactors[FACTOR_TRACKBALL] = 0f
    // Adjust the values if we want to enablerotation by default.
    mFactors[FACTOR_ROTATION] = 10.0f
    mFactors[FACTOR_NAV] = 5.0f
    mFactors[FACTOR_MAJORNAV] = 5.0f
    mFactors[FACTOR_SYSOPS] = 5.0f
    mFactors[FACTOR_APPSWITCH] = 5.0f
    mFactors[FACTOR_FLIP] = 0f
    // disbale permission by default
    mFactors[FACTOR_PERMISSION] = 5.0f
    mFactors[FACTOR_ANYTHING] = 10.0f
    mFactors[FACTOR_PINCHZOOM] = 10.0f
}
```
### generateActivity()如下
```kotlin
fun generateActivity() {
    val e = MonkeyActivityEvent(mMainApp[mRandom.nextInt(mMainApps.size)])
    mQ.addLast(e)
}
```
- 将用从getMainApps里得到的可以启动的app中随机的一个声明一个MonkeyActivityEvent的对象,并加入到MonkeyEventQueue mQ中
```kotlin
class MonkeyActivityEvent(private val mApp: ComponentName, public val mAlarmTime: Long = 0) :MonkeyEvent(EVENT_TYPE_ACTIVITY)
```

### 该类的validate函数及其被调用函数为
```kotlin
override fun validate(): Boolean {
    var ret = true
    // only populate & dump permissions if enabled
    if (mFactors[FACTOR_PERMISSION] != 0.0f) {
        ret = ret and mPermissionUtil.populatePermissionsMapping()
        if (ret && mVerbose >= 2) {
            mPermissionUtil.dump()
        }
    }
    return ret and adjustEventFactors()
}

...

private fun adjustEventFactors(): Boolean {
    // go through all values and compute totals for user & default values
    val (userSum, defaultSum, defaultCount) = mFactors.fold(Triple(0f, 0f, 0)){ 
        (userFold, defaultFold, defaultCount), 
        factor -> 
        if (factor <= 0f) { Triple(userFold-factor, defaultFold, defaultCount)}
        else {Triple(userFold, defaultFold+factor, defaultCount+1)}
    }
    // if the user request was > 100%, reject it
    if (userSum > 100f){
        Logger.lPrintln("** Event weights > 100%")
        return false
    }
    //if the user specified all of the weights, they need to be 100%
    if (defaultCount == 0 && (userSum < 99.9f || userSum > 100.1f)) {
        Logger.lPrintln("** Event weight != 100%")
        return false
    }
    // compute the adjustment necessary
    var defaultsTarget = 100f - userSum
    val defaultsAdjustment = defaultsTarget / defaultSum
    //fix all values, by adjusting defaults, or flipping user values back to >0
    for (i in 0 until FACTORZ_COUNT) {
        mFactors[i] = if (mFactors[i] <= 0f) {-mFactors[i]}
        else {mFactors[i] * defaultsAdjustment}
    }
    // if verbose show factors
    if (mVerbose > 0) {
        Logger.lPrintln("// Event percentages:")
        mFactors.forEachIndexed { index, fl ->
            Logger.lPrintln("// $index: $fl%")
        }
    }
    if (!validateKeys()) {
        return false
    }
    //finally, normalize and convert to running sum
    var sum = 0f
    for (i in 0 until FACTORZ_COUNT) {
        sum += mFactors[i] / 100f
        mFactors[i] = sum
    }
    return true
}
```
- adjustEventFactors 统计当前用户设定的比例userSum和剩余的系统初始化的比例defaultSum和剩余系统初始化事件比例的个数defaultCount.
- 统计完后判断是否符合不会超过100等限制。
- 然后将defaultSum等比例缩放到100 - userSum，使得两者相加为100.
- <font color=red size=3>validatekeys</font> 用于检查一些key比如home,back之类的
- 最后进行归一化并采取0-1区间划分的方法.



### getNextEvent()
- 在函数runMonkeyCycle中会调用该函数来获取下一个注入事件。
```kotlin
override fun getNextEvent(): MonkeyEvent {
    if (mQ.isEmpty()) {
        generateEvents()
    }
    mEventCount++
    val e = mQ.first
    mQ.removeFirst()
    return e
}
```
- 如果mQ不为空，那么以为这mQ中还有由generateActivity()得到的event.(即还有可以启动的app)
- 如果mQ为空，意味着需要启动的app都已经启动了，然后可以去生成一些压力测试的事件.

### generateEvents()
- 在getNextEvent()中被调用
```kotlin
private fun generateEvents() {
    val cls = mRandom.nextFloat()
    var lastKey: Int
    when{
        //触摸
        cls < mFactors[FACTOR_TOUCH] ->{
            println("Touch")
            generatePointerEven(mRandom, GESTURE_TAP)
            return
        }
        //手势
        cls < mFactors[FACTOR_MOTION]-> {
            println("Motion")
            generatePointerEven(mRandom, GESTURE_DRAG)
            return
        }
        //二指缩放
        cls < mFactor[FACTOR_PINCHZOOM] -> {
            println("PINCHZOOM")
            generatePointerEven(mRandom,GESTURE_PINCH_OR_ZOOM)
            return
        }
        //轨迹球(已被弃用)
        cls < mFactor[FACTOR_TRACKBALL] -> {
            println("TrackBall")
            generateTrackballEven(mRandom)
            return
        }
        //旋转
        cls < mFactors[FACTOR_ROTATION]-> {
            println("Rotation")
            generateRotationEven(mRandom)
            return
        }
        //请求权限
        cls  < mFactor[FACTOR_PERMISSION] -> {
            println("Permission")
            val permissionEvent =mPermissionUtilgenerateRandomPermissionEvet(mRandom)
            if (permissionEvent ==null) {
                if (mVerbose > 1) {
                    Logger.lPrintl("WARNING: Unableto generatepermission event")
                }
                // no permission eventcan be generated,generate a touch eventinstead
                generatePointerEven(mRandom, GESTURE_TAP)
                return
            }
            mQ.add(permissionEvent)
            return
        }
    }
    ......
}
```
- 随机生成一个事件
- <font color=red size=3>TBD</font>

### random类函数(随机生成点，偏移量和随机移动)
```kotlin
private fun randomPoint(random: Random, display: Display): PointF {
    // since Display.getWidth() is deprecated, usethe recommanded way; x is width and y is height
    val size = Point()
    display.getSize(size)
    return PointF(random.nextInt(size.x).toFloat(),random.nextInt(size.y).toFloat())
}
private fun randomVector(random: Random) =
    PointF((random.nextFloat() - 0.5f) * 50,(random.nextFloat() - 0.5f) * 50)
private fun randomWalk(random: Random, display:Display, point: PointF, vector: PointF) {
    val size = Point()
    //获取屏幕的宽(存入size.x)和高(存入size.y)
    display.getSize(size)
    //max和min是防止移动超过范围
    point.x = Math.max(Math.min(point.x + randomnextFloat() * vector.x, size.x.toFloat()), 0f)
    point.y = Math.max(Math.min(point.y + randomnextFloat() * vector.y, size.y.toFloat()), 0f)
}
```


### generatePointerEvent
- 用来生成Touch, Motion, PinchZoom事件
```kotlin
private fun generatePointerEvent(random: Random, gesture: Int) {
    //获取默认显示器
    val display = DisplayManagerGlobal.getInstance(.getRealDisplay(Display.DEFAULT_DISPLAY)
    //随机生成点和偏移量
    val p1 = randomPoint(random, display)
    val v1 = randomVector(random)
    //从开机到现在的毫秒数
    val downAt = SystemClock.uptimeMillis()
    //第一个手指按下
    mQ.addLast(
        MonkeyTouchEvent(MotionEvent.ACTION_DOWN)setDownTime(downAt).addPointer(0, p1.x, p1y)
            .setIntermediateNote(false)
    )
    // sometimes we'll move during the touch
    if (gesture == GESTURE_DRAG) {
        repeat(random.nextInt(10)) {
            randomWalk(random, display, p1, v1)
            //println("add MOVE")
            mQ.addLast(
                MonkeyTouchEvent(MotionEventACTION_MOVE).setDownTime(downAt)addPointer(0, p1.x, p1.y)
                    .setIntermediateNote(true)
            )
           // println((mQ.last as MonkeyMotionEvent.getAction())
        }
    }
    else if (gesture == GESTURE_PINCH_OR_ZOOM) {
        val p2 = randomPoint(random, display)
        val v2 = randomVector(random)
        randomWalk(random, display, p1, v1)
        mQ.addLast(
            MonkeyTouchEvent(
                //Action_POINTER_DOWN 表示有非主要指按下即之前已经有手指在屏幕上
                //or == |
                // 1 shl index_shift 表示这个actio的index是1， 使用第8到15位作为index
                MotionEvent.ACTION_POINTER_DOWN or(1 shl MotionEventACTION_POINTER_INDEX_SHIFT)
            ).setDownTime(downAt)
                .addPointer(0, p1.x, p1.y)addPointer(1, p2.x, p2.y)setIntermediateNote(true)
        )
        repeat(random.nextInt(10)) {
            randomWalk(random, display, p1, v1)
            randomWalk(random, display, p2, v2)
            mQ.addLast(
                MonkeyTouchEvent(MotionEventACTION_MOVE).setDownTime(downAt)addPointer(0, p1.x, p1.y)
                    .addPointer(1, p2.x, p2.y)setIntermediateNote(true)
            )
        }
        randomWalk(random, display, p1, v1)
        randomWalk(random, display, p2, v2)
        //第二个手指离开
        mQ.addLast(
            MonkeyTouchEvent(MotionEventACTION_POINTER_UP or (1 shl MotionEventACTION_POINTER_INDEX_SHIFT))
                .setDownTime(downAt).addPointer(0,p1.x, p1.y).addPointer(1, p2.x, p2y)
                .setIntermediateNote(true)
        )
    }
    //randomwalk后位置保存在p1中
    randomWalk(random, display, p1, v1)
    //第一个手指离开
    mQ.addLast(
        MonkeyTouchEvent(MotionEvent.ACTION_UP)setDownTime(downAt).addPointer(0, p1.x, p1y)
            .setIntermediateNote(false)
    )
}
```
- <font color=red size=3> TBD</font>

<font color=red size=5> TBD </font>