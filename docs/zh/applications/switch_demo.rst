.. _switch_demo:

涂鸦云开关示例（switch_demo）
============================

概述
----

``switch_demo`` 是一个简单的、跨平台、跨系统、支持多种连接方式的开关示例应用。通过涂鸦APP、涂鸦云服务，用户可以对这个开关进行远程控制（外出）、局域网控制（同一局域网）和蓝牙控制（没有可用网络）。

.. image:: ../../../images/zh/switch_demo_overview.png
   :width: 600px
   :alt: 开关示例概览

目录结构
--------

``switch_demo`` 的目录结构如下：

.. code-block:: bash

    +- switch_demo
        +- libqrencode        # 二维码工具库
        +- src
            -- cli_cmd.c      # 命令行操作接口
            -- qrencode_print.c  # 二维码显示工具
            -- tuya_main.c    # 主要功能实现
            -- tuya_config.h  # 涂鸦PID和授权信息配置
        -- CMakeLists.txt     # 编译配置文件
        -- README_CN.md       # 中文说明文档
        -- README.md          # 英文说明文档

工作原理
--------

``switch_demo`` 基于涂鸦IoT平台，实现了一个智能开关的基本功能。其核心工作原理如下：

1. 设备初始化
~~~~~~~~~~~~~

设备启动后，首先进行硬件初始化，包括GPIO、网络接口等基础设施的初始化。然后读取授权信息，准备连接涂鸦云平台。

.. code-block:: c

    // 初始化涂鸦IoT SDK
    tuya_iot_init(&client, &config);
    
    // 注册事件回调函数
    tuya_iot_register_event_cb(&client, event_handler);

2. 网络连接
~~~~~~~~~~~

设备支持多种网络连接方式：

- **Wi-Fi连接**：通过Wi-Fi连接到互联网，实现远程控制
- **蓝牙连接**：在无网络环境下，通过蓝牙直接控制设备
- **有线网络连接**：通过以太网连接到互联网

设备会根据当前可用的网络环境，自动选择合适的连接方式。

3. 设备配网与绑定
~~~~~~~~~~~~~~~~

设备首次使用时，需要进行配网和绑定操作：

- **Wi-Fi配网**：通过涂鸦APP引导用户输入Wi-Fi信息
- **蓝牙配网**：通过蓝牙直接传输网络信息
- **二维码配网**：对于有线网络，生成二维码供APP扫描绑定

.. code-block:: c

    // 生成并显示二维码
    qrencode_print_to_terminal(url);

4. 状态管理与控制
~~~~~~~~~~~~~~~~

设备通过DP点（Data Point）管理状态，主要包括：

- **开关状态**：控制设备的开/关
- **工作模式**：设置设备的工作模式
- **运行状态**：反馈设备当前的运行状态

当用户通过APP发送控制指令时，设备会接收到对应的DP点变更，并执行相应的操作。

.. code-block:: c

    // DP点处理函数
    void tuya_iot_dp_process(tuya_iot_client_t *client, const tuya_iot_dp_t *dp_array, size_t dp_cnt)
    {
        // 处理开关状态变更
        if (dp_array[i].id == SWITCH_DP_ID) {
            bool state = dp_array[i].value.value_bool;
            // 执行开关操作
            set_switch_state(state);
            // 上报执行结果
            report_switch_state(client, state);
        }
    }

5. 状态上报
~~~~~~~~~~

设备会定期或在状态变更时，主动向云端上报当前状态：

- **定时上报**：定期向云端同步设备状态
- **变更上报**：当设备状态发生变化时，立即上报
- **查询上报**：响应云端的状态查询请求

.. code-block:: c

    // 上报开关状态
    void report_switch_state(tuya_iot_client_t *client, bool state)
    {
        tuya_iot_dp_t dp;
        dp.id = SWITCH_DP_ID;
        dp.type = TUYA_IOT_DP_TYPE_BOOL;
        dp.value.value_bool = state;
        tuya_iot_dp_report_json(client, &dp, 1);
    }

工作流程
--------

``switch_demo`` 的完整工作流程如下：

1. 设备上电与初始化
~~~~~~~~~~~~~~~~~

- 硬件初始化
- SDK初始化
- 读取授权信息
- 注册事件回调

2. 网络连接阶段
~~~~~~~~~~~~~

- 检测可用网络
- 根据网络环境选择连接方式
- 建立与涂鸦云的连接

3. 配网与绑定阶段
~~~~~~~~~~~~~~~

- 首次使用时进入配网模式
- 通过APP引导用户完成配网
- 设备与用户账号绑定

4. 正常工作阶段
~~~~~~~~~~~~~

- 接收云端/APP控制指令
- 执行相应操作
- 上报设备状态
- 响应用户本地操作

5. 异常处理
~~~~~~~~~

- 网络异常重连
- 云端连接恢复
- 本地模式降级

使用说明
--------

1. 环境准备
~~~~~~~~~

在使用 ``switch_demo`` 前，请确保已完成以下准备工作：

- 安装涂鸦智能APP
- 注册涂鸦开发者账号
- 在涂鸦IoT平台创建产品并获取PID
- 获取TuyaOpen授权码

2. 配置修改
~~~~~~~~~

修改 ``tuya_config.h`` 文件，填入您的产品信息：

.. code-block:: c

    // 产品PID
    #define TUYA_PRODUCT_KEY "xxxxxxxxxxxxxxxx"
    
    // TuyaOpen授权码
    #define TUYA_DEVICE_UUID "uuidxxxxxxxxxxxxxxxx"
    #define TUYA_DEVICE_AUTHKEY "keyxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"

3. 编译与烧录
~~~~~~~~~~~

.. code-block:: bash

    # 选择目标开发板
    tos config_choice
    
    # 配置项目
    tos menuconfig
    
    # 编译项目
    tos build
    
    # 烧录到设备
    tos flash

4. 设备配网
~~~~~~~~~

- **Wi-Fi配网**：打开涂鸦APP，选择"添加设备"，按照提示完成配网
- **蓝牙配网**：确保手机蓝牙已开启，通过APP发现并配置设备
- **有线网络**：设备会在串口或显示屏上显示二维码，使用APP扫描完成绑定

5. 设备控制
~~~~~~~~~

- **远程控制**：通过涂鸦APP在任何有网络的地方控制设备
- **局域网控制**：在同一局域网下，APP可直接控制设备，无需经过云端
- **蓝牙控制**：在设备附近，可通过蓝牙直接控制设备

6. 命令行操作
~~~~~~~~~~~

设备支持以下命令行操作：

.. code-block:: bash

    # 查看设备信息
    switch info
    
    # 切换开关状态
    switch toggle
    
    # 查看当前状态
    switch status
    
    # 重置设备
    switch reset

接口说明
--------

``switch_demo`` 主要使用以下涂鸦IoT SDK接口：

1. 初始化接口
~~~~~~~~~~~

.. code-block:: c

    /**
     * @brief 初始化涂鸦IoT客户端
     * @param client 客户端实例
     * @param config 配置参数
     * @return 成功返回OPRT_OK
     */
    tuya_iot_init(tuya_iot_client_t *client, const tuya_iot_config_t *config);

2. 事件处理接口
~~~~~~~~~~~~~

.. code-block:: c

    /**
     * @brief 注册事件回调函数
     * @param client 客户端实例
     * @param cb 回调函数
     * @return 成功返回OPRT_OK
     */
    tuya_iot_register_event_cb(tuya_iot_client_t *client, tuya_iot_event_cb_t cb);

3. 连接管理接口
~~~~~~~~~~~~~

.. code-block:: c

    /**
     * @brief 连接涂鸦云
     * @param client 客户端实例
     * @return 成功返回OPRT_OK
     */
    tuya_iot_connect(tuya_iot_client_t *client);
    
    /**
     * @brief 断开与涂鸦云的连接
     * @param client 客户端实例
     * @return 成功返回OPRT_OK
     */
    tuya_iot_disconnect(tuya_iot_client_t *client);

4. 数据点操作接口
~~~~~~~~~~~~~~~

.. code-block:: c

    /**
     * @brief 上报数据点
     * @param client 客户端实例
     * @param dp_array 数据点数组
     * @param dp_cnt 数据点数量
     * @return 成功返回OPRT_OK
     */
    tuya_iot_dp_report_json(tuya_iot_client_t *client, const tuya_iot_dp_t *dp_array, size_t dp_cnt);

常见问题
--------

1. 设备无法配网
~~~~~~~~~~~~~

- 检查Wi-Fi信号强度
- 确认APP权限设置
- 验证产品PID配置是否正确

2. 设备无法连接云端
~~~~~~~~~~~~~~~~

- 检查网络连接
- 验证授权码是否正确
- 确认设备时间是否同步

3. 控制指令无响应
~~~~~~~~~~~~~~

- 检查设备在线状态
- 验证DP点ID配置
- 查看设备日志输出

4. 状态上报异常
~~~~~~~~~~~~

- 检查上报接口调用
- 验证数据格式是否正确
- 确认网络连接稳定性

总结
----

``switch_demo`` 提供了一个完整的智能开关解决方案，通过涂鸦IoT平台，实现了设备的远程控制、状态管理和多种连接方式支持。开发者可以基于此示例，快速开发自己的智能设备产品。

.. note::
   本文档基于TuyaOpen最新版本编写，如有变更请以最新代码为准。