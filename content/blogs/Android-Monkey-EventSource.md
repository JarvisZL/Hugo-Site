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
- 该类的validate函数及其被调用函数为
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
- <font color=red size=3>validatekeys</font> 
- 最后进行归一化并采取0-1区间划分的方法.