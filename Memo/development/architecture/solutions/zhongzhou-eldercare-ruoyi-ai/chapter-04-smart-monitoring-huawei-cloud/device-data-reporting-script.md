---
type: memo
date: 2026-11-06
aliases:
  - "she-bei-shu-ju-shang-bao-jiao-ben"
  - "设备数据上报脚本"
---

# 设备数据上报脚本

## 智能定位手表

```Java
/*
 * Copyright (c) 2020-2023 Huawei Cloud Computing Technology Co., Ltd. All rights reserved.
 *
 * Redistribution and use in source and binary forms, with or without modification,
 * are permitted provided that the following conditions are met:
 *
 * 1. Redistributions of source code must retain the above copyright notice, this list of
 *    conditions and the following disclaimer.
 *
 * 2. Redistributions in binary form must reproduce the above copyright notice, this list
 *    of conditions and the following disclaimer in the documentation and/or other materials
 *    provided with the distribution.
 *
 * 3. Neither the name of the copyright holder nor the names of its contributors may be used
 *    to endorse or promote products derived from this software without specific prior written
 *    permission.
 *
 * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS
 * "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO,
 * THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR
 * PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR
 * CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL,
 * EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO,
 * PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS;
 * OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY,
 * WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR
 * OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF
 * ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
 */

package com.huaweicloud.sdk.iot.device.demo.device;

import com.huaweicloud.sdk.iot.device.IoTDevice;
import com.huaweicloud.sdk.iot.device.client.requests.ServiceProperty;
import com.huaweicloud.sdk.iot.device.transport.ActionListener;
import org.apache.logging.log4j.LogManager;
import org.apache.logging.log4j.Logger;

import java.io.File;
import java.io.IOException;
import java.io.InputStream;
import java.nio.file.Files;
import java.security.SecureRandom;
import java.util.Arrays;
import java.util.HashMap;
import java.util.Map;
import java.util.Random;

import static java.nio.file.StandardCopyOption.REPLACE_EXISTING;

/**
 * 演示如何直接使用DeviceClient进行设备属性的上报和读写
 */
public class PropertySampleWatch {

    private static final String IOT_ROOT_CA_RES_PATH = "ca.jks";

    private static final String IOT_ROOT_CA_TMP_PATH = "huaweicloud-iotda-tmp-" + IOT_ROOT_CA_RES_PATH;

    private static final Logger log = LogManager.getLogger(PropertySampleWatch.class);

    public static void main(String[] args) throws InterruptedException, IOException {

        // 加载iot平台的ca证书，进行服务端校验
        File tmpCAFile = new File(IOT_ROOT_CA_TMP_PATH);
        try (InputStream resource = CommandSample.class.getClassLoader().getResourceAsStream(IOT_ROOT_CA_RES_PATH)) {
            Files.copy(resource, tmpCAFile.toPath(), REPLACE_EXISTING);
        }

        // 创建设备并初始化. 用户请替换为自己的接入地址。
        IoTDevice device = new IoTDevice("ssl://{地址替换成自己的地址}:8883",
            "{设备Id替换成自己的}", "{设备密钥替换成自己的}", tmpCAFile);
        if (device.init() != 0) {
            return;
        }

        // 定时上报属性
        while (true) {

            Map<String, Object> json = new HashMap<>();
            Random rand = new SecureRandom();

            // 按照物模型设置属性
            json.put("BodyTemp", 36);
            json.put("xueyang", rand.nextFloat() * 100.0f);
            json.put("HeartRate", rand.nextFloat() * 100.0f);
            json.put("BatteryPercentage", rand.nextFloat() * 100.0f);

            ServiceProperty serviceProperty = new ServiceProperty();
            serviceProperty.setProperties(json);
            serviceProperty.setServiceId("{替换成自己的}"); // serviceId要和物模型一致

            device.getClient().reportProperties(Arrays.asList(serviceProperty), new ActionListener() {
                @Override
                public void onSuccess(Object context) {
                    log.info("pubMessage success");
                }

                @Override
                public void onFailure(Object context, Throwable var2) {
                    log.error("reportProperties failed" + var2.toString());
                }
            });
            Thread.sleep(10000);
        }
    }
}
```



## 烟雾报警器

```Java
/*
 * Copyright (c) 2020-2023 Huawei Cloud Computing Technology Co., Ltd. All rights reserved.
 *
 * Redistribution and use in source and binary forms, with or without modification,
 * are permitted provided that the following conditions are met:
 *
 * 1. Redistributions of source code must retain the above copyright notice, this list of
 *    conditions and the following disclaimer.
 *
 * 2. Redistributions in binary form must reproduce the above copyright notice, this list
 *    of conditions and the following disclaimer in the documentation and/or other materials
 *    provided with the distribution.
 *
 * 3. Neither the name of the copyright holder nor the names of its contributors may be used
 *    to endorse or promote products derived from this software without specific prior written
 *    permission.
 *
 * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS
 * "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO,
 * THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR
 * PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR
 * CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL,
 * EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO,
 * PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS;
 * OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY,
 * WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR
 * OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF
 * ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
 */

package com.huaweicloud.sdk.iot.device.demo.device;

import com.huaweicloud.sdk.iot.device.IoTDevice;
import com.huaweicloud.sdk.iot.device.client.requests.ServiceProperty;
import com.huaweicloud.sdk.iot.device.transport.ActionListener;
import org.apache.logging.log4j.LogManager;
import org.apache.logging.log4j.Logger;

import java.io.File;
import java.io.IOException;
import java.io.InputStream;
import java.nio.file.Files;
import java.util.Arrays;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

import static java.nio.file.StandardCopyOption.REPLACE_EXISTING;

/**
 * 演示如何直接使用DeviceClient进行设备属性的上报和读写
 */
public class PropertySampleSmoke {

    private static final String IOT_ROOT_CA_RES_PATH = "ca.jks";

    private static final String IOT_ROOT_CA_TMP_PATH = "huaweicloud-iotda-tmp-" + IOT_ROOT_CA_RES_PATH;

    private static final Logger log = LogManager.getLogger(PropertySampleSmoke.class);

    public static void main(String[] args) throws InterruptedException, IOException {

        // 加载iot平台的ca证书，进行服务端校验
        File tmpCAFile = new File(IOT_ROOT_CA_TMP_PATH);
        try (InputStream resource = CommandSample.class.getClassLoader().getResourceAsStream(IOT_ROOT_CA_RES_PATH)) {
            Files.copy(resource, tmpCAFile.toPath(), REPLACE_EXISTING);
        }

        // 创建设备并初始化. 用户请替换为自己的接入地址。
        IoTDevice device = new IoTDevice("ssl://{替换成自己的地址}:8883",
            "{替换成自己的设备id}", "{替换成自己的设备密钥}", tmpCAFile);
        if (device.init() != 0) {
            return;
        }

        // 定时上报属性
        while (true) {

            Map<String, Object> json = new HashMap<>();
            List<Integer> currentHumidityList =Arrays.asList(21,22,23,24,25,26,27,28,29,11);
            List<Integer>indoorTemperatureList =Arrays.asList(22,12,23,24,25,26,27,28,29,18);
            // 按照物模型设置属性
            json.put("CurrentHumidity", currentHumidityList.get((int)(Math.random() * currentHumidityList.size())));
            json.put("IndoorTemperature", indoorTemperatureList.get((int)(Math.random() * indoorTemperatureList.size())));
            json.put("SmokeSensorState", 0);

            ServiceProperty serviceProperty = new ServiceProperty();
            serviceProperty.setProperties(json);
            serviceProperty.setServiceId("{替换成自己的}"); // serviceId要和物模型一致

            device.getClient().reportProperties(Arrays.asList(serviceProperty), new ActionListener() {
                @Override
                public void onSuccess(Object context) {
                    log.info("pubMessage success");
                }

                @Override
                public void onFailure(Object context, Throwable var2) {
                    log.error("reportProperties failed" + var2.toString());
                }
            });
            Thread.sleep(10000);
        }
    }
}
```



## 睡眠监测带

```Java
/*
 * Copyright (c) 2020-2023 Huawei Cloud Computing Technology Co., Ltd. All rights reserved.
 *
 * Redistribution and use in source and binary forms, with or without modification,
 * are permitted provided that the following conditions are met:
 *
 * 1. Redistributions of source code must retain the above copyright notice, this list of
 *    conditions and the following disclaimer.
 *
 * 2. Redistributions in binary form must reproduce the above copyright notice, this list
 *    of conditions and the following disclaimer in the documentation and/or other materials
 *    provided with the distribution.
 *
 * 3. Neither the name of the copyright holder nor the names of its contributors may be used
 *    to endorse or promote products derived from this software without specific prior written
 *    permission.
 *
 * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS
 * "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO,
 * THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR
 * PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR
 * CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL,
 * EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO,
 * PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS;
 * OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY,
 * WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR
 * OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF
 * ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
 */

package com.huaweicloud.sdk.iot.device.demo.device;

import com.huaweicloud.sdk.iot.device.IoTDevice;
import com.huaweicloud.sdk.iot.device.client.requests.ServiceProperty;
import com.huaweicloud.sdk.iot.device.transport.ActionListener;
import org.apache.logging.log4j.LogManager;
import org.apache.logging.log4j.Logger;

import java.io.File;
import java.io.IOException;
import java.io.InputStream;
import java.nio.file.Files;
import java.security.SecureRandom;
import java.util.*;

import static java.nio.file.StandardCopyOption.REPLACE_EXISTING;

/**
 * 演示如何直接使用DeviceClient进行设备属性的上报和读写
 */
public class PropertySampleSleep {

    private static final String IOT_ROOT_CA_RES_PATH = "ca.jks";

    private static final String IOT_ROOT_CA_TMP_PATH = "huaweicloud-iotda-tmp-" + IOT_ROOT_CA_RES_PATH;

    private static final Logger log = LogManager.getLogger(PropertySampleSleep.class);

    public static void main(String[] args) throws InterruptedException, IOException {

        // 加载iot平台的ca证书，进行服务端校验
        File tmpCAFile = new File(IOT_ROOT_CA_TMP_PATH);
        try (InputStream resource = CommandSample.class.getClassLoader().getResourceAsStream(IOT_ROOT_CA_RES_PATH)) {
            Files.copy(resource, tmpCAFile.toPath(), REPLACE_EXISTING);
        }

        // 创建设备并初始化. 用户请替换为自己的接入地址。
        IoTDevice device = new IoTDevice("ssl://{替换成自己的地址}:8883",
                "{替换成自己的设备id}", "{替换成自己的设备密钥}", tmpCAFile);
        if (device.init() != 0) {
            return;
        }

        // 定时上报属性
        while (true) {

            Map<String, Object> json = new HashMap<>();
            Random rand = new SecureRandom();

            List<Integer> bedExitCountList = Arrays.asList(1, 2, 3, 4, 5, 4, 3, 2, 1);
//            List<Integer> heartRateList =Arrays.asList(87,88,77,87,90,100,76,67,87);
            List<Integer> heartRateList = Arrays.asList(56, 60, 55, 60, 60, 63, 61, 62, 61);
            List<Integer> respiratoryRateList = Arrays.asList(87, 88, 77, 87, 90, 100, 76, 67, 87);

            // 按照物模型设置属性
            json.put("BedTime", new Date());
            json.put("BedExitCount", bedExitCountList.get((int) (Math.random() * bedExitCountList.size())));
            json.put("SleepPhaseState", 2);
            json.put("HeartRate", heartRateList.get((int) (Math.random() * heartRateList.size())));
            json.put("RespiratoryRate", respiratoryRateList.get((int)(Math.random() *respiratoryRateList.size())));

            ServiceProperty serviceProperty = new ServiceProperty();
            serviceProperty.setProperties(json);
            serviceProperty.setServiceId("{替换成自己的serviceId}"); // serviceId要和物模型一致

            device.getClient().reportProperties(Arrays.asList(serviceProperty), new ActionListener() {
                @Override
                public void onSuccess(Object context) {
                    log.info("pubMessage success");
                }

                @Override
                public void onFailure(Object context, Throwable var2) {
                    log.error("reportProperties failed" + var2.toString());
                }
            });
            Thread.sleep(10000);
        }
    }
}

```

---
> 来源：飞书技术知识库（脱敏版） · 07 业务组件方案集 / 中州养老项目（若依+AI） / 第四章-智能监测系统（华为云） / 设备数据上报脚本 · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/E2aAwKCXdiFOFfkI12pcYVKAnCg)
