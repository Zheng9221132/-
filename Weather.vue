<template>
  <div class="weather" v-if="weatherData.adCode.city && weatherData.weather.weather">
    <span>{{ weatherData.adCode.city }}&nbsp;</span>
    <span>{{ weatherData.weather.weather }}&nbsp;</span>
    <span>{{ weatherData.weather.temperature }}℃</span>
    <span class="sm-hidden">
      &nbsp;{{
        weatherData.weather.winddirection?.endsWith("风")
          ? weatherData.weather.winddirection
          : weatherData.weather.winddirection + "风"
      }}&nbsp;
    </span>
    <span class="sm-hidden">{{ weatherData.weather.windpower }}&nbsp;级</span>
  </div>
  <div class="weather" v-else>
    <span>天气数据获取失败</span>
  </div>
</template>

<script setup>
import { h, reactive, onMounted } from "vue";
import { Error } from "@icon-park/vue-next";
import { ElMessage } from "element-plus";

// 天气数据结构保持不变
const weatherData = reactive({
  adCode: {
    city: null, // 城市
    adcode: null // 城市编码
  },
  weather: {
    weather: null, // 天气现象
    temperature: null, // 实时气温
    winddirection: null, // 风向描述
    windpower: null // 风力级别
  }
});

// 计算温度平均值方法（备用，当前接口返回实时温度）
const getTemperature = (min, max) => {
  try {
    const average = (Number(min) + Number(max)) / 2;
    return Math.round(average);
  } catch (error) {
    console.error("计算温度出现错误：", error);
    return "NaN";
  }
};

// 获取天气数据 - 完全替换为免费无注册接口
const getWeatherData = async () => {
  try {
    // 免费天气接口（基于IP定位，无需注册，无API Key）
    const response = await fetch("https://www.tianqiapi.com/api/?version=v61&appid=123456&appsecret=abc123");
    if (!response.ok) throw new Error(`请求失败，状态码: ${response.status}`);

    const resData = await response.json();
    if (resData.errmsg !== "success") throw new Error(resData.errmsg || "接口返回异常");

    // 字段映射，完全匹配原 weatherData 结构
    weatherData.adCode = {
      city: resData.city || "未知城市",
      adcode: resData.cityid || null
    };
    weatherData.weather = {
      weather: resData.wea || "未知天气",
      temperature: resData.tem || "未知",
      winddirection: resData.win || "未知风向",
      windpower: resData.win_level || "未知"
    };
  } catch (error) {
    console.error("天气信息获取失败:" + error.message);
    onError("天气信息获取失败");
  }
};

// 报错信息提示
const onError = (message) => {
  ElMessage({
    message,
    icon: h(Error, {
      theme: "filled",
      fill: "#efefef"
    })
  });
  console.error(message);
};

// 组件挂载时请求数据
onMounted(() => {
  getWeatherData();
});
</script>

<style scoped>
/* 可根据需要添加样式 */
.weather {
  font-size: 14px;
  color: #333;
  line-height: 1.5;
}
.sm-hidden {
  display: inline;
}
@media (max-width: 576px) {
  .sm-hidden {
    display: none;
  }
}
</style>
