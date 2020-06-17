---
author: "Lingyu Zhang"
author_link: "https://jarviszly.com"
title: "Android Monkey GetMainApps"
date: 2020-06-16T16:37:05+08:00
lastmod: 2020-06-16T16:37:05+08:00
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

## getMainApps函数

### 作用
- 根据命令中-p和-c参数来在设备中寻找符合条件的Activities.


### 代码分析
```kotlin
try{
    mMainCategories.forEach {
        if (it == null) return@forEach
        val intent = Intent(Intent.ACTION_MAIN)
        if (it.isNotEmpty()) {
            intent.addCategory(it)
        }
        val mainApps = mPm!!.queryIntentActivities(intent, null, 0, UserHandle.myUserId()) 
        ...      
    }
    ...
}
```
- 上述代码设置隐式的Intent,然后通过mpm查询设备中符合Intent这些属性的Activity放入mainApps中。
- 此处mainApps实际上是一个list\<ResolveInfo\>

```kotlin
try{
    ...
    mainApps.forEach { r ->
        val packageName = r.activityInfoapplicationInfo.packageName
        //查看对应的包名是否在mValidPackages中或者mValidPackages为空
        if (MonkeyUtils.sFiltercheckEnteringPackage(packageName)) {
            if (mVerbose >= 2 ) {// very verbose
                Logger.lPrintln("// + Using main activity ${r.  activityInfo.name} from package " +
                        "$packageName)")
            }
            //记录下app的uid和构建一个显示的ComponentName(用于之后启动对应的activity)
            mUids.add(r.activityInfo.applicationInfo.uid)
            mMainApps.add(ComponentName(packageName, r. activityInfo.name))
        } else{
            if (mVerbose >= 3) { //very very verbose
                Logger.lPrintln("// - NOT USING main activity ${r.  activityInfo.name} from package" +
                        "$packageName)")
            }
        }
    }
    ...
}

```

```kotlin
//当得到可以启动的app不为空时，返回true.
return if (mMainApps.isEmpty()) {
            Logger.lPrintln("** No activities found to run monkey aborted")
            false
} else true
```