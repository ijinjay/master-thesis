# UE4 tango 插件
采用官方提供的Tango C SDK文件进行开发。

## 配置Tango的构建脚本
在`TangoPlugin.Build.cs`中


```
using UnrealBuildTool;
using System.IO;
public class TangoPlugin : ModuleRules {
    public TangoPlugin(TargetInfo Target) {

        // 配置Tango插件的include路径
        PublicIncludePaths.Add("TangoPlugin/Public");
        PrivateIncludePaths.Add("TangoPlugin/Private");

        // 指定插件依赖的系统模块
        PublicDependencyModuleNames.AddRange(new string[ {
            "Core",
            "CoreUObject",
            "Engine",
            "RenderCore",
            "ShaderCore",
            "RHI"
        });
        PrivateDependencyModuleNames.AddRange(new string[] {
            "CoreUObject",
            "Engine",
            "RHI",
            "RenderCore",
            "Core"
        });

        // 添加到系统插件的设置菜单
        PrivateIncludePathModuleNames.Add("Settings");

        // 在Android平台下配置额外的规则来编译和使用Tango API
        if (Target.Platform == UnrealTargetPlatform.Android) {
            // 添加对启动模块的依赖
            PrivateDependencyModuleNames.Add("Launch");
            // 添加额外的属性配置文件
            AdditionalPropertiesForReceipt.Add(new ReceiptProperty("AndroidPlugin", Path.Combine(ModuleDirectory, "TangoPlugin_APL.xml")));
            // 输出系统架构
            System.Console.WriteLine("android arch: "+Target.Architecture);
            // 添加tango c API include目录
            PublicIncludePaths.Add(Path.Combine(ModuleDirectory, "../../ThirdParty", "tango_client_api", "include"));
            PublicIncludePaths.Add(Path.Combine(ModuleDirectory, "../../ThirdParty", "tango_support_api", "include"));
            PublicIncludePaths.Add(Path.Combine(ModuleDirectory, "../../ThirdParty", "tango_3d_reconstruction", "include"));
            // 添加tango c API lib目录
            foreach (string ArchDir in new string[] { "armeabi-v7a", "x86", "arm64-v8a" })
            {
                PublicLibraryPaths.Add(Path.Combine(ModuleDirectory, "../../ThirdParty", "tango_client_api", "lib", ArchDir));
                PublicLibraryPaths.Add(Path.Combine(ModuleDirectory, "../../ThirdParty", "tango_support_api", "lib", ArchDir));
                PublicLibraryPaths.Add(Path.Combine(ModuleDirectory, "../../ThirdParty", "tango_3d_reconstruction", "lib", ArchDir));
            }
            // 添加tango c API lib文件
            PublicAdditionalLibraries.Add("tango_client_api");
            PublicAdditionalLibraries.Add("tango_support_api");
            PublicAdditionalLibraries.Add("tango_3d_reconstruction");
        }
    }
}
```

额外的属性配置文件中:

```
<?xml version="1.0" encoding="utf-8"?>
<root xmlns:android="http://schemas.android.com/apk/res/android">
    <trace enable="true"/>
    <!-- TangoPlugin插件 Android构建初始化配置 -->
    <init>
        <log text="TangoPlugin Plugin Build Initialisation"/>
        <!-- 读取DefaultEngine.ini文件来确定是否需要Tango权限 -->
        <setBoolFromProperty result="bTangoAreaLearningEnabled" ini="$S(PluginDir)/../../../../Config/DefaultEngine" section="/Script/TangoPlugin.TangoRuntimeSettings" property="bTangoAreaLearningEnabled" default="true" />
    </init>

    <!-- 添加额外的Tango设置到AndroidManifest.xml -->
    <androidManifestUpdates>
      <log text="TangoPlugin Plugin Android Manifest Additions"/>
        <setElement result="bTangoAreaLearningEnabledElement" value="meta-data" />
        <addAttribute tag="$bTangoAreaLearningEnabledElement" name="android:name" value="com.projecttango.bTangoAreaLearningEnabled" />
        <addAttribute tag="$bTangoAreaLearningEnabledElement" name="android:value" value="$B(bTangoAreaLearningEnabled)"/>
        <addElement tag="application" name="bTangoAreaLearningEnabledElement" />
    <setElement result="bRequiresTangoLibraryElement" value="uses-library"/>
    <addAttribute tag="$bRequiresTangoLibraryElement" name="android:name" value="com.projecttango.libtango_device2"/>
    <addAttribute tag="$bRequiresTangoLibraryElement" name="android:required" value="true"/>
    <addElement tag="application" name="bRequiresTangoLibraryElement" />

    <!-- 配置摄像机权限 -->
    <addPermission android:name="android.permission.CAMERA"/>

    <!-- 配置导入和导出ADF文件 -->
    <addPermission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
    <addPermission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
    <addElements tag="application">
    </addElements>
    </androidManifestUpdates>

    <!-- 拷贝项目的Java文件，即TangoInterface类文件;.so库文件 -->
    <resourceCopies>
    <log text="TangoPlugin Plugin Android Source Code and Library Resource Copies"/>
    <log text="PluginDir: $S(PluginDir)"/>
    <copyDir src="$S(PluginDir)/../Java/com" dst="$S(BuildDir)/src/com" />
    <isArch arch="armeabi-v7a">
        <copyFile src="$S(PluginDir)/../../ThirdParty/tango_support_api/lib/armeabi-v7a/libtango_support_api.so"
                        dst="$S(BuildDir)/libs/armeabi-v7a/libtango_support_api.so" />
        <copyFile src="$S(PluginDir)/../../ThirdParty/tango_3d_reconstruction/lib/armeabi-v7a/libtango_3d_reconstruction.so"
      dst="$S(BuildDir)/libs/armeabi-v7a/libtango_3d_reconstruction.so" />
        </isArch>
        <isArch arch="arm64-v8a">
        <copyFile src="$S(PluginDir)/../../ThirdParty/tango_support_api/lib/arme64-v8a/libtango_support_api.so"
                        dst="$S(BuildDir)/libs/arm64-v8a/libtango_support_api.so" />
        <copyFile src="$S(PluginDir)/../../ThirdParty/tango_3d_reconstruction/lib/arm64-v8a/libtango_3d_reconstruction.so"
      dst="$S(BuildDir)/libs/arm64-v8a/libtango_3d_reconstruction.so" />
        </isArch>
        <isArch arch="x86">
            <copyFile src="$S(PluginDir)/../../ThirdParty/tango_support_api/lib/x86/libtango_support_api.so"
                        dst="$S(BuildDir)/libs/x86/libtango_support_api.so" />
            <copyFile src="$S(PluginDir)/../../ThirdParty/tango_3d_reconstruction_/lib/x86/libtango_3d_reconstruction.so"
      dst="$S(BuildDir)/libs/x86/libtango_3d_reconstruction.so" />
        </isArch>
    </resourceCopies> 
  
    <!-- 导入Tango Interface 类到GameActivity中, 此处引用的Java模块必须被拷贝 -->
    <gameActivityImportAdditions>
        <insert>
            import com.projecttango.plugin.TangoInterface;
            import android.util.Log;
        </insert>
    </gameActivityImportAdditions>

    <!-- 添加一个Tango Interface的实例到GameActivity -->
    <gameActivityClassAdditions>
        <insert>
      // Tango 接口实现
      private TangoInterface tangoInterface;

      public Context AndroidThunkJava_GetAppContext()
      {
        return this.getApplicationContext();
      }

      public void AndroidThunkJava_RequestTangoService()
      {
        if(tangoInterface != null)
        {
            tangoInterface.RequestTangoService(this);
        }
        return;
      }
      
      public void AndroidThunkJava_UnbindTangoService()
      {
        if(tangoInterface != null)
        {
            tangoInterface.UnbindTangoService(this);
        }
        return;
      }
      
      public void AndroidThunkJava_RequestImportPermission(String Filename)
      {
        if(tangoInterface != null)
        {
          tangoInterface.requestImportPermissions(this, Filename);
        }
        return;
      }

      public void AndroidThunkJava_RequestExportPermission(String UUID, String Filename)
      {
        if(tangoInterface != null)
        {
         tangoInterface.requestExportPermissions(this, UUID, Filename);
        }
      return;
      }
    </insert>
    </gameActivityClassAdditions>

    <!-- 确定Tango接口是否可以被创建，及需要运行哪一个intent -->
    <gameActivityReadMetadataAdditions>
        <insert>
            boolean bIsTangoAreaLearningEnabled = false;
            // 是否Area learning开启
            if (bundle.containsKey("com.projecttango.bTangoAreaLearningEnabled"))
            {
                bIsTangoAreaLearningEnabled = bundle.getBoolean("com.projecttango.bTangoAreaLearningEnabled");
                Log.debug( "Found bTangoAreaLearningEnabled = " + bIsTangoAreaLearningEnabled);
            }
                tangoInterface = new TangoInterface(bIsTangoAreaLearningEnabled);
            
        </insert>
    </gameActivityReadMetadataAdditions>

    <!-- Activity挂起时通知tango挂起 -->
    <gameActivityOnResumeAdditions>
        <insert>
            if (tangoInterface != null)
            {
                tangoInterface.resume(this);
            }
        </insert>
    </gameActivityOnResumeAdditions>

    <!-- Activity暂停时暂停Tango -->
  <gameActivityOnPauseAdditions>
    <insert>
      if (tangoInterface != null) {
        tangoInterface.pause();
      }
    </insert>
  </gameActivityOnPauseAdditions>
  <!-- 传递ActivityResults给Tango -->
    <gameActivityOnActivityResultAdditions>
        <insert>
            if (tangoInterface != null &amp;&amp; tangoInterface.handleActivityResult(requestCode, resultCode, data, this))
            {
                Log.debug("Tango Interface handled onActivityResult");
            }
        </insert>
    </gameActivityOnActivityResultAdditions>
    
    <!-- 在开启GameActivity前加载静态库文件 -->
    <soLoadLibrary>
        <insert>
            final int ARCH_ERROR = -2;
            final int ARCH_FALLBACK = -1;
            final int ARCH_DEFAULT = 0;
            final int ARCH_ARM64 = 1;
            final int ARCH_ARM32 = 2;
            final int ARCH_X86_64 = 3;
            final int ARCH_X86 = 4;

            Log.debug("TangoVMTests: Shim Static class called.");
            Log.debug("TangoLifecycleDebugging: Now attempting to load the shared library.");
            int loadedSoId = ARCH_ERROR;
            String basePath = "/data/data/com.google.tango/libfiles/";
            if (!(new File(basePath).exists())) {
            basePath = "/data/data/com.projecttango.tango/libfiles/";
            }
            Log.debug("TangoInitializationHelper: basePath: " + basePath);
            try {
            System.load(basePath + "arm64-v8a/libtango_client_api.so");
            loadedSoId = ARCH_ARM64;
            Log.debug("TangoInitializationHelper: Success! Using arm64-v8a/libtango_client_api.");
            } catch (UnsatisfiedLinkError e) {
            }
            //Note: &lt; = lessthan operator
            if (loadedSoId &lt; ARCH_DEFAULT) {
            try {
            System.load(basePath + "armeabi-v7a/libtango_client_api.so");
            loadedSoId = ARCH_ARM32;
            Log.debug("TangoInitializationHelper: Success! Using armeabi-v7a/libtango_client_api.");
            } catch (UnsatisfiedLinkError e) {
            }
            }
            if (loadedSoId &lt; ARCH_DEFAULT) {
            try {
            System.load(basePath + "x86_64/libtango_client_api.so");
            loadedSoId = ARCH_X86_64;
            Log.debug("TangoInitializationHelper: Success! Using x86_64/libtango_client_api.");
            } catch (UnsatisfiedLinkError e) {
            }
            }
            if (loadedSoId &lt; ARCH_DEFAULT) {
            try {
            System.load(basePath + "x86/libtango_client_api.so");
            loadedSoId = ARCH_X86;
            Log.debug("TangoInitializationHelper: Success! Using x86/libtango_client_api.");
            } catch (UnsatisfiedLinkError e) {
            }
            }
            if (loadedSoId &lt; ARCH_DEFAULT) {
            try {
            System.load(basePath + "default/libtango_client_api.so");
            loadedSoId = ARCH_DEFAULT;
            Log.debug("TangoInitializationHelper: Success! Using default/libtango_client_api.");
            } catch (UnsatisfiedLinkError e) {
            }
            }
            if (loadedSoId &lt; ARCH_DEFAULT) {
            try {
            System.loadLibrary("tango_client_api");
            loadedSoId = ARCH_FALLBACK;
            Log.debug("Falling back to libtango_client_api.so symlink.");
            } catch (UnsatisfiedLinkError e) {
            }
            }
            
            // 确保加载库文件成功
            int LibraryLoadResult = loadedSoId;
            if (LibraryLoadResult == ARCH_ERROR)
            {
            Log.debug("TangoLifecycleDebugging: ERROR! Unable to load libtango_client_api.so!");
            }
            else
            {
            Log.debug("TangoLifecycleDebugging: Success! Able to load libtango_client_api.so!");
            }
            if(LibraryLoadResult == ARCH_FALLBACK)
            {
            Log.debug("TangoLifecycleDebugging: NOTE: Unable to load libtango_client_api.so from new source. Fell back to the Symlink version. Will not work outside a Yellowstone tablet!");
            }
        </insert>
    </soLoadLibrary>
</root>
```

## ITangoAR 接口







