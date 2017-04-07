# Android 传感器数据获取

传感器类别：

> 运动传感器

运动传感器测量三轴角速度和加速度。包括加速计，重力传感器和旋转向量传感器。

> 环境传感器

测量环境的参数，比如环境的温度、气压、光照和湿度。包括气压计，光度计和温度计

> 定位传感器

测量设备的物理位置。包括方向传感器和磁感器。


传感器分为基于硬件和基于软件。基于硬件的传感器是设备上的物理组件。基于软件的传感器不是物理传感器，而是从一个或多个硬件传感器中解析数据。

Android平台的传感器类型列表：

传感器|类型|描述|使用场景
:---|:----|:----|:----
TYPE_ACCELEROMETER | 硬件 | 测量加速度$m/s^2$，包括重力 | 运动检测
TYPE_AMBIENT_TEMPERATURE | 硬件 | 测量环境温度℃ | 监测空气温度
TYPE_GRAVITY | 软件或硬件 | 测量设备的重力$m/s^2$ | 运动检测
TYPE_GYROSCOPE | 硬件| 测量设备的旋转率$rad/s$ | 旋转检测
TYPE_LIGHT | 硬件 | 测量环境光等级$lx$ | 控制屏幕亮度
TYPE_LINEAR_ACCELERATION | 软件或硬件 | 测量加速度，去除重力的影响 | 监控设备在某一个轴上的运动
TYPE_MAGNETIC_FIELD | 硬件 | 测量环境的磁力$\mu T$ | 创建指南针应用
TYPE_ORIENTATION | 软件 | 测量设备在三轴的旋转角度 | 确定设备的位置
TYPE_PRESSURE | 硬件 | 测量空气压强$hPa$| 监测压强变化
TYPE_ROTATION_VECTOR | 软件或硬件 | 提供设备的选择向量来测量设备的方向 | 运动检测和选择检测
TYPE_TEMPERATURE | 硬件 | 测量设备的温度℃ | 监测设备温度变化


使用Android 传感器框架来获取数据，这个框架在`android.hardware`包中提供。包括

`SensorManager`:用于创建传感器服务实例。
`Sensor`:用于创建特定传感器实例。
`SensorEvent`:创建事件对象。
`SensorEventListener`:创建回调函数来接收传感器事件。

## 确保传感器是否可以使用
首先获取传感器服务对象，通过调用`getSystemService()`来获取`SensorManager`，然后传递`SENSOR_SERVICE`参数：

```Java
private SensorManager mSensorManager;
...
mSensorManager = (SensorManager) getSystemService(Context.SENSOR_SERVICE);
```

然后，获取传感器列表:

```Java
List<Sensor> deviceSensors = mSensorManager.getSensorList(Sensor.TYPE_ALL);
```

示例：

```
private SensorManager mSensorManager;
private Sensor mSensor;

...

mSensorManager = (SensorManager) getSystemService(Context.SENSOR_SERVICE);
mSensor = null;

if (mSensorManager.getDefaultSensor(Sensor.TYPE_GRAVITY) != null){
  List<Sensor> gravSensors = mSensorManager.getSensorList(Sensor.TYPE_GRAVITY);
  for(int i=0; i<gravSensors.size(); i++) {
    if ((gravSensors.get(i).getVendor().contains("Google Inc.")) &&
       (gravSensors.get(i).getVersion() == 3)){
      // Use the version 3 gravity sensor.
      mSensor = gravSensors.get(i);
    }
  }
}
if (mSensor == null){
  // Use the accelerometer.
  if (mSensorManager.getDefaultSensor(Sensor.TYPE_ACCELEROMETER) != null){
    mSensor = mSensorManager.getDefaultSensor(Sensor.TYPE_ACCELEROMETER);
  }
  else{
    // Sorry, there are no accelerometers on your device.
    // You can't play this game.
  }
}
```

## 监测传感器事件
通过两个回调函数来检测传感器数据:`onAccuracyChanged()`和`onSensorChanged()`。Android系统在下面事件发生时调用：
1. 传感器精度变化，系统调用`onAccuracyChanged()`，精度表示为四种状态：`SENSOR_STATUS_ACCURACY_LOW`, `SENSOR_STATUS_ACCURACY_MEDIUM`, `SENSOR_STATUS_ACCURACY_HIGH`或`SENSOR_STATUS_ACCURACY_UNRELIABLE`。
2. 传感器报告新值，系统调用`onSensorChanged()`，并提供`SensorEvent`对象。 

```Java
public class SensorActivity extends Activity implements SensorEventListener {
  private SensorManager mSensorManager;
  private Sensor mLight;

  @Override
  public final void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.main);

    mSensorManager = (SensorManager) getSystemService(Context.SENSOR_SERVICE);
    mLight = mSensorManager.getDefaultSensor(Sensor.TYPE_LIGHT);
  }

  @Override
  public final void onAccuracyChanged(Sensor sensor, int accuracy) {
    // Do something here if sensor accuracy changes.
  }

  @Override
  public final void onSensorChanged(SensorEvent event) {
    // The light sensor returns a single value.
    // Many sensors return 3 values, one for each axis.
    float lux = event.values[0];
    // Do something with this sensor value.
  }

  @Override
  protected void onResume() {
    super.onResume();
    mSensorManager.registerListener(this, mLight, SensorManager.SENSOR_DELAY_NORMAL);
  }

  @Override
  protected void onPause() {
    super.onPause();
    mSensorManager.unregisterListener(this);
  }
}
```

## 传感器坐标系




























