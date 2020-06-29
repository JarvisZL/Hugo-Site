# Android Monkey Events


## MonkeyActivityEvent


## MonkeyMoitionEvent
- 是一个抽象类
```kotlin
public abstract class MonkeyMotionEvent protected constructor
    (type: Int, private val mSource : Int, 
    //the Type and mSource must be specified, as it defines what type the event is (e.g., EVENT_TYPE_TOUCH)
    //and to which source (e.g, screen or trackball), others can be mocked
    private val mAction: Int, private var mDownTime: Long = -1L,
    private var mEventTime: Long = -1L, private var mXPrecision : Float = 1f,
    private var mYPrecision : Float = 1f) : MonkeyEvent(type){
        private val mPointers = SparseArray<MotionEvent.PointerCoords>()

        // If true, this is an intermediate step (more verbose logging, only)
        private var mIntermediateNote: Boolean = false

        private var mMetaState = 0
        private var mDeviceId: Int = 0
        private val mFlags: Int = 0
        private var mEdgeFlags: Int = 0
        

        fun addPointer(id: Int, x: Float, y: Float) = addPointer(id, x, y, 0F, 0F)
        open fun addPointer(id: Int, x: Float, y: Float, pressure: Float, size: Float)  = apply {
            val c = MotionEvent.PointerCoords()
            c.x = x
            c.y = y
            c.pressure = pressure
            c.size = size
            mPointers.append(id, c)
        }
        ......
}
```
- set类函数: 都是用this.apply执行，返回当前对象。
```kotlin
fun setIntermediateNote(b: Boolean) = this.apply { mIntermediateNote = b }
fun setDownTime(downTime: Long) = this.apply { mDownTime = downTime }
fun setEventTime(eventTime: Long) = this.apply {mEventTime = eventTime }
fun setMetaState(metaState : Int) = this.apply {mMetaState = metaState }
fun setPrecision(xPrecision: Float, yPrecision:Float) = this.apply {
    mXPrecision = xPrecision
    mYPrecision = yPrecision
}
fun setDeviceId(deviceId: Int) = this.apply {mDeviceId = deviceId }
fun setEdgeFlags(edgeFlags: Int) = this.apply {mEdgeFlags = edgeFlags }
```
- get类函数
```kotlin
fun getIntermediateNote() = mIntermediateNote
fun getAction() = mAction
fun getDownTime() = mDownTime
fun getEventTime() = mEventTime
```
- event相关函数
```kotlin
internal fun getEvent(): MotionEvent {
    val pointerCount = mPointers.size()
    val pointerIds = IntArray(pointerCount){mPointers.keyAt(it) }
    val pointerCoords = Array<MotionEventPointerCoords>(pointerCount){mPointers.valueA(it)}
    return MotionEvent.obtain(mDownTime, if(mEventTime < 0) SystemClock.uptimeMillis()else mEventTime,
        mAction, pointerCount, pointerIds,pointerCoords, mMetaState, mXPrecision,mYPrecision, mDeviceId, mEdgeFlags,mSource, mFlags
    )
}
override fun isThrottlable() = mAction ==MotionEvent.ACTION_UP
override fun injectEvent(iwm: IWindowManager, iam:IActivityManager, verbose: Int): Int {
    //println("Inject Event! $mAction")
    val me = getEvent()
    //println("Inject Event! ${me.action}  ${meactionMasked}")
    if ((verbose > 0 /*&& !mIntermediateNote*/) ||verbose > 1) {
        val points = StringBuilder()
        for (i in 0 until me.pointerCount) {
            points.append(" ${me.getPointerId(i)}:({me.getX(i)},${me.getY(i)})")
        }
        Logger.lPrintln(":Sending ${getTypeLabel()}(${
            when(me.actionMasked) {
                MotionEvent.ACTION_DOWN ->"ACTION_DOWN"
                MotionEvent.ACTION_MOVE ->"ACTION_MOVE"
                MotionEvent.ACTION_UP   ->"ACTION_UP"
                MotionEvent.ACTION_CANCEL ->"ACTION_CANCEL"
                MotionEvent.ACTION_POINTER_DOWN ->"ACTION_POINTER_DOWN ${megetPointerId(me.actionIndex)}"
                MotionEvent.ACTION_POINTER_UP ->"ACTION_POINTER_UP ${me.getPointerI(me.actionIndex)}"
                else -> mAction
            }
        }):$points")
    }
    try {
        if (!InputManager.getInstance()injectInputEvent(me,
                InputManagerINJECT_INPUT_EVENT_MODE_WAIT_FOR_FIISH)) {return INJECT_FAIL}
    } finally {
        me.recycle()
    }
    return INJECT_SUCCESS
}
```


## MonkeyTouchEvent
- EventSource中generatePointEvent会实例化一个匿名的MonkeyTouchEvent的对象然后调用其一些函数
```kotlin
class MonkeyTouchEvent(action: Int) :
    MonkeyMotionEvent(EVENT_TYPE_TOUCH, InputDevice.SOURCE_TOUCHSCREEN, action) {

    override fun getTypeLabel() = "Touch"

    override fun addPointer(id: Int, x: Float, y: Float, pressure: Float, size: Float): MonkeyMotionEvent {
        var _y = y
        if (_y < statusBarHeight) { // avoid touch status bar
            _y = (statusBarHeight + 1).toFloat()
        }
        return super.addPointer(id, x, _y, pressure, size)
    }

    companion object {

        internal val statusBarHeight: Int

        //The status bar is 24 dp, and the statusBarHeight is its actual px
        init {
            val display = DisplayManagerGlobal.getInstance().getRealDisplay(Display.DEFAULT_DISPLAY)
            val dm = DisplayMetrics()
            display.getMetrics(dm)
            statusBarHeight = (24 * dm.density).toInt()
        }
    }
}
```
