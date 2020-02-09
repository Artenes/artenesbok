---
title: "Using Robolectric for Unit Tests in Android"
date: 2019-06-24T21:01:34-04:00
draft: false
tags: 
  - android
  - test
---

To start, add the following to your ``build.gradle`` file:
````groovy
android  {  
  testOptions  {  
    unitTests  {  
      includeAndroidResources  =  true  
    }  
  }  
}  

dependencies  {
  testImplementation 'org.robolectric:robolectric:4.3'
}
````

And annotate your test with the Robolectric test runner:
````kotlin
import org.robolectric.RobolectricTestRunner

@RunWith(RobolectricTestRunner::class)  
public class SandwichTest {
}
````

We want to test the following class. It represents the device and exposes two methods: ``ìsInAirplaneMode`` and ``ìsWithLowBattery``. Both of them make use of the Android framework to check if the device is with Airplane mode enabled or if it has enough battery.

````kotlin
import android.content.Context
import android.provider.Settings
import android.provider.Settings.Global.AIRPLANE_MODE_ON
import android.os.BatteryManager

public class Device(private val context: Context) {  
      
  fun isInAirplaneMode(): Boolean {  
    return Settings.System.getInt(context.contentResolver, Settings.Global.AIRPLANE_MODE_ON, 0) != 0
  }  

  fun isWithLowBattery(): Boolean {  
    val manager = context.getSystemService(Context.BATTERY_SERVICE) as BatteryManager  
    val level = manager.getIntProperty(BatteryManager.BATTERY_PROPERTY_CAPACITY)  
    val isCharging = manager.isCharging
    return level <= 15 && !isCharging  
  }  
    
}
````

## To test ``isInAirplaneMode`` method
We would have two tests to validate the two possible outcomes, both implemented in the code bloack below. Pay attention to the ``ShadowSettings`` class. It is a class provided by Robolectric and it belogs to this collection of classes called "Sadow" classes. Instances of these classes are created along the real Android system classes to allow you to manipulate their behaviour. Such as in this test case where we call ``ShadowSettings.setAirplaneMode`` to set if either true or false would be returned from the ``Settings.System.getInt`` call in our ``Device`` class to check if Airplane is enabled or not.

We can figure it out if an Android class has a Shadow available in Robolectric by prepending "Shadow" to the name of the class. We can use the [Robolectric API docs](http://robolectric.org/javadoc/4.3/) to see if such a class exists or not by typing its name in the search box at the top of the page. If it does not exists, we would have to implement it ourselves, which is not complicated.

````kotlin
import org.robolectric.shadows.ShadowSettings
import org.junit.Before  
import org.junit.Test
import org.robolectric.RuntimeEnvironment

@Before  
fun setUp() {  
  val context = RuntimeEnvironment.application  
  device = Device(context)  
}  
  
@Test  
fun isInAirplaneMode_returnsTrueIfAirplaneIsEnabled() {  
  //given the device is with airplane on  
  ShadowSettings.setAirplaneMode(true)  

  //when I check it  
  val isAirplaneOn = device.isInAirplaneMode()  

  //then it should be true  
  assertTrue(isAirplaneOn)  
}  
  
@Test  
fun isInAirplaneMode_returnsFalseIfAirplaneIsDisabled() {  
  //given the device is with airplane off  
  ShadowSettings.setAirplaneMode(false)  

  //when I check it  
  val isAirplaneOn = device.isInAirplaneMode()  

  //then it should be false  
  assertFalse(isAirplaneOn)  
}
````

## To test ``isWithLowBattery`` method
Same thing here, two tests will be necessary. The difference is in the ``setUp`` method. Since we we are dealing with a non-static method, we have to get the reference to the shadow object that we need to manipulate. Inside of ``Device`` class we get the instance of ``BatteryManager`` by calling ``context.getSystemService(Context.BATTERY_SERVICE) as BatteryManager``. We will do the same, but we will pass the reference to the Robolectric method called ``Shadows.shadowOf``. As said before, each object has a corresponding shadow, so to get it we need the object reference and a call to ``Shadows.shadowOf``. It works like a key-value map, where the reference is the key.

After getting the reference, we can manipulate it by calling the methods that it provided. In this case I need to control the level of the battery and if either the device is charging or not.

````kotlin
import org.junit.Before  
import org.junit.Test
import org.robolectric.RuntimeEnvironment
import org.robolectric.shadows.ShadowBatteryManager
import android.os.BatteryManager

private lateinit var device: Device  
private lateinit var batteryManager: ShadowBatteryManager  
  
@Before  
fun setUp() {  
  val context = RuntimeEnvironment.application  
  device = Device(context)  
  batteryManager = Shadows.shadowOf(context.getSystemService(Context.BATTERY_SERVICE) as BatteryManager)  
}
  
@Test  
fun isWithLowBattery_returnsTrueIfDeviceIsWithLowBatteryAndNotCharging() {  
  //given the device has 10% of battery and it is not charging  
  batteryManager.setIntProperty(BatteryManager.BATTERY_PROPERTY_CAPACITY, 10)  
  batteryManager.setIsCharging(false)  

  //when I check it  
  val isLowBattery = device.isWithLowBattery()  

  //then it should be true  
  assertTrue(isLowBattery)  
}  
  
@Test  
fun isWithLowBattery_returnsFalseIfDeviceIsWithBatteryAndIsCharging() {  
  //given the device has 90% of battery and it is charging  
  batteryManager.setIntProperty(BatteryManager.BATTERY_PROPERTY_CAPACITY, 90)  
  batteryManager.setIsCharging(true)  

  //when I check it  
  val isLowBattery = device.isWithLowBattery()  

  //then it should be false  
  assertFalse(isLowBattery)  
}
````

## Conclusion
This is the basic gist of Robolectric. Instead of using ``mocks``, we use ``fakes`` to test out code that depends on Android. Robolectric provides a ton of shadows for various Android classes. But if there is a class that do not has a shadow, [you can easily implement a shadow for it](https://himbeer.farm/2018/11/custom-shadows/).