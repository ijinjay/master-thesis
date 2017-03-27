
# Tango开启彩色相机和深度图像

```C++
  // 配置Tango服务的运行方式
  tango_config_ = TangoService_getConfig(TANGO_CONFIG_DEFAULT);
  if (tango_config_ == nullptr) {
    LOGE("Failed to get default config form");
    std::exit(EXIT_SUCCESS);
  }

  // 开启彩色相机
  int ret = TangoConfig_setBool(tango_config_, "config_enable_color_camera", true);
  if (ret != TANGO_SUCCESS) {
    LOGE("config_enable_color_camera() failed with error code: %d", ret);
    std::exit(EXIT_SUCCESS);
  }
  // 注册回调函数
  ret = TangoService_connectOnFrameAvailable(TANGO_CAMERA_COLOR, this, OnFrameAvailableRouter);
  if (ret != TANGO_SUCCESS) {
    LOGE("Error connecting color frame %d", ret);
    std::exit(EXIT_SUCCESS);
  }


  // 开启深度相机
  TangoErrorType err = TangoConfig_setBool(tango_config_, "config_enable_depth", true);
  if (err != TANGO_SUCCESS) {
    LOGE( "config_enable_depth() failed with error code: %d.", err);
    std::exit(EXIT_SUCCESS);
  }
  // 需要指定深度相机的数据格式为XYZC
  ret = TangoConfig_setInt32(tango_config_, "config_depth_mode", TANGO_POINTCLOUD_XYZC);
  if (ret != TANGO_SUCCESS)
  {
    LOGE("Failed to set 'depth_mode' configuration flag with error code: %d", ret);
    std::exit(EXIT_SUCCESS);
  }
  // 注册回调函数
  err = TangoService_connectOnPointCloudAvailable(OnPointCloudAvailable);
  if (err != TANGO_SUCCESS) {
    LOGE("Failed to connect to point cloud callback with error code: %d", err);
    std::exit(EXIT_SUCCESS);
  }

  // 连接到tango服务
  ret = TangoService_connect(this, tango_config_);
  if (ret != TANGO_SUCCESS) {
    LOGE("Failed to connect to the Tango service with error code: %d", ret);
    std::exit(EXIT_SUCCESS);
  }

  // Initialize TangoSupport context.
  // 初始化Tango context
  TangoSupport_initializeLibrary();
```

