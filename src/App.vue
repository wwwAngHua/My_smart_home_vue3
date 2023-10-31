<script setup lang="ts">
import bg from './assets/bg.jpg'
import { ref } from 'vue'

const client_id = ref('650bd657da0243b282d9cab6d75a80ff')

// 设备在线状态标识
const status = ref<string>('正在连接')
// 设备在线状态类型（颜色）
const status_type = ref<string>('warning')
// 灯泡状态
const status_led = ref<number>(0)

// 温度
const temperature = ref<number>(0)
// 湿度
const humidity = ref<number>(0)

// 温度进度条颜色变化
const temperature_colors = [
    { color: '#337ecc', percentage: 10 },
    { color: '#409EFF', percentage: 20 },
    { color: '#67C23A', percentage: 30 },
    { color: '#F56C6C', percentage: 40 },
    { color: '#FF0000', percentage: 100 },
]

// 湿度进度条颜色变化
const humidity_colors = [
    { color: '#f56c6c', percentage: 20 },
    { color: '#e6a23c', percentage: 40 },
    { color: '#5cb87a', percentage: 60 },
    { color: '#1989fa', percentage: 80 },
    { color: '#6f7ad3', percentage: 100 },
]

const tip = ref<string>('正在连接设备。')

// 设备在线状态监控
const _status = ref<number>(0)
setInterval(function () {
    _status.value++
    if (_status.value > 10) {
        console.log('设备离线')
        status.value = '离线'
        status_type.value = 'danger'
        tip.value = '设备离线，请接通设备电源。'
    }
}, 1000)

// 使用 Paho MQTT （EMQX Public Server）
const clientID = Math.round(Math.random() * 80 + 20)
const client = new Paho.MQTT.Client(
    'broker.emqx.io',
    Number(8083),
    'My-smart-home' + clientID,
)

// 设置回调处理程序
client.onConnectionLost = onConnectionLost // 重新连接
client.onMessageArrived = onMessageArrived // 接收消息

// 建立 MQTT 连接
client.connect({ onSuccess: onConnect })

// 客户端连接时调用
function onConnect() {
    console.log('连接成功')
    status.value = '连接成功'
    status_type.value = 'success'
    tip.value = '设备连接成功，等待设备响应。'
    // 订阅主题
    client.subscribe('wanghua/my/smart/home/' + client_id.value)
}

// 当客户端失去连接时调用
function onConnectionLost(responseObject: any) {
    if (responseObject.errorCode !== 0) {
        console.log('连接丢失: ' + responseObject.errorMessage)
        status.value = '连接丢失'
        status_type.value = 'warning'
        tip.value = '连接丢失，请尝试刷新当前页面。'
    }
}

// 消息到达时调用
function onMessageArrived(message: any) {
    _status.value = 0
    const data = JSON.parse(message.payloadString)
    console.log(data)
    status_led.value = parseInt(data.led_state)
    status.value = '在线'
    status_type.value = 'success'
    temperature.value = parseFloat(data.temperature)
    humidity.value = parseFloat(data.humidity)
    let tip_text: string = ''
    if (Number(data.temperature) < 22) {
        tip_text += '温度过低，建议打开空调或取暖器，'
    } else if (Number(data.temperature) > 26) {
        tip_text += '温度过高，建议打开窗户通风，'
    } else {
        tip_text += '温度适宜，'
    }
    if (Number(data.humidity) < 50) {
        tip_text += '湿度过低，建议打开加湿器。'
    } else if (Number(data.humidity) > 60) {
        tip_text += '湿度过高，建议打开空调或窗户通风。'
    } else {
        tip_text += '湿度适宜，Have a nice day！'
    }
    tip.value = tip_text
}

// 开关灯泡
function led() {
    message = new Paho.MQTT.Message('0')
    message.destinationName = 'wanghua/my/smart/home/control/' + client_id.value // 主题
    client.send(message)
    if (status_led.value == 1) {
        //layer.msg('灯泡已关闭')
    } else {
        //layer.msg('灯泡已打开')
    }
}
</script>

<template>
    <div :style="{ 'background-image': `url(${bg})` }" class="container">
        <div class="centered-div">
            <el-badge :value="status" :type="status_type"
                ><el-text style="color: white" class="ms-title"
                    >My smart home by
                    <el-text size="small" style="color: white"
                        >王华</el-text
                    ></el-text
                ></el-badge
            >
            <div style="margin-top: 20px">
                <el-progress
                    :width="110"
                    type="circle"
                    :percentage="temperature"
                    :color="temperature_colors"
                    style="
                        margin-right: 10px;
                        border-radius: 99px;
                        background: rgba(255, 255, 255, 0.3);
                    "
                    ><template #default="{ percentage }">
                        <el-text style="color: white"
                            >{{ percentage }}°C</el-text
                        >
                        <br />
                        <el-text style="color: white">温度</el-text>
                    </template></el-progress
                >
                <el-progress
                    :width="110"
                    type="circle"
                    :percentage="humidity"
                    :color="humidity_colors"
                    style="
                        margin-left: 10px;
                        border-radius: 99px;
                        background: rgba(255, 255, 255, 0.3);
                    "
                    ><template #default="{ percentage }">
                        <el-text style="color: white"
                            >{{ percentage }}%</el-text
                        >
                        <br />
                        <el-text style="color: white">湿度</el-text>
                    </template></el-progress
                >
            </div>
            <div>
                <svg
                    v-if="status_led == 0"
                    @click="led"
                    t="1669783127299"
                    viewBox="0 0 1024 1024"
                    version="1.1"
                    xmlns="http://www.w3.org/2000/svg"
                    p-id="16388"
                    width="23"
                    height="23"
                    style="
                        margin-top: 10px;
                        padding-left: 10px;
                        padding-right: 10px;
                        border-radius: 99px;
                        background: rgba(255, 255, 255, 0.3);
                    ">
                    >
                    <path
                        d="M575.1808 829.1328c17.5616 0 31.8208 14.1312 31.8208 31.5904 0 16.1024-12.16 29.4144-28.0064 31.3344l-3.2 0.256h-125.6704a31.6928 31.6928 0 0 1-31.7952-31.5904c0-16.128 12.1344-29.44 27.904-31.36l3.8912-0.256h125.056z m34.2016-71.0656c17.5616 0 31.7952 14.1312 31.7952 31.5904 0 16.128-12.1344 29.44-27.9808 31.36l-3.2 0.2304h-195.3792a31.6928 31.6928 0 0 1-31.7952-31.5904c0-16.1024 12.1344-29.44 27.9808-31.3344l3.2-0.256h195.3792zM512 131.712c164.864 0 298.5216 132.5056 298.5216 296.0128 0 97.8176-48.2048 187.4944-127.1808 242.432l-5.0688 3.4048-6.4512 29.696a57.088 57.088 0 0 1-50.7136 44.4416l-4.4032 0.256h-209.792a57.1392 57.1392 0 0 1-55.1168-41.4976l-0.9984-4.1472-5.12-24.4736-1.2032-5.0944-5.2992-3.6096a295.04 295.04 0 0 1-125.184-223.8208l-0.384-8.7808-0.128-8.8064C213.4784 264.2176 347.136 131.712 512 131.712z m0 63.1552c-129.7664 0-234.9312 104.2688-234.9312 232.8576a231.7824 231.7824 0 0 0 96.0768 187.8528l6.528 4.608 1.9712 1.28c10.6496 7.0656 15.4368 13.0048 20.4544 23.9616 0.896 1.9456 1.6896 3.968 2.3552 6.016l1.536 5.1968 1.3824 5.632 2.56 11.8784 2.176 10.5984h198.656l5.8112-26.624 4.2752-20.352 2.7648-4.0448 17.4848-11.4176c66.0736-43.1872 105.8304-115.8656 105.8304-194.56 0-128.6144-105.1648-232.8832-234.9312-232.8832z m-54.3232 49.7408a31.7952 31.7952 0 0 1 37.12 25.2416c3.2 17.152-8.192 33.664-25.4464 36.864-56.96 10.5216-83.0208 44.544-86.784 112.4608a31.744 31.744 0 0 1-33.536 29.7984 31.6672 31.6672 0 0 1-30.0032-33.28c5.376-96.3584 51.584-154.9824 138.6496-171.0848z"
                        p-id="16389"
                        fill="#ffffff"></path>
                </svg>
                <svg
                    v-else
                    @click="led"
                    t="1669783127299"
                    viewBox="0 0 1024 1024"
                    version="1.1"
                    xmlns="http://www.w3.org/2000/svg"
                    p-id="16388"
                    width="23"
                    height="23"
                    style="
                        margin-top: 10px;
                        padding-left: 10px;
                        padding-right: 10px;
                        border-radius: 99px;
                        background: rgba(255, 255, 255, 0.3);
                    ">
                    >
                    <path
                        d="M575.1808 829.1328c17.5616 0 31.8208 14.1312 31.8208 31.5904 0 16.1024-12.16 29.4144-28.0064 31.3344l-3.2 0.256h-125.6704a31.6928 31.6928 0 0 1-31.7952-31.5904c0-16.128 12.1344-29.44 27.904-31.36l3.8912-0.256h125.056z m34.2016-71.0656c17.5616 0 31.7952 14.1312 31.7952 31.5904 0 16.128-12.1344 29.44-27.9808 31.36l-3.2 0.2304h-195.3792a31.6928 31.6928 0 0 1-31.7952-31.5904c0-16.1024 12.1344-29.44 27.9808-31.3344l3.2-0.256h195.3792zM512 131.712c164.864 0 298.5216 132.5056 298.5216 296.0128 0 97.8176-48.2048 187.4944-127.1808 242.432l-5.0688 3.4048-6.4512 29.696a57.088 57.088 0 0 1-50.7136 44.4416l-4.4032 0.256h-209.792a57.1392 57.1392 0 0 1-55.1168-41.4976l-0.9984-4.1472-5.12-24.4736-1.2032-5.0944-5.2992-3.6096a295.04 295.04 0 0 1-125.184-223.8208l-0.384-8.7808-0.128-8.8064C213.4784 264.2176 347.136 131.712 512 131.712z m0 63.1552c-129.7664 0-234.9312 104.2688-234.9312 232.8576a231.7824 231.7824 0 0 0 96.0768 187.8528l6.528 4.608 1.9712 1.28c10.6496 7.0656 15.4368 13.0048 20.4544 23.9616 0.896 1.9456 1.6896 3.968 2.3552 6.016l1.536 5.1968 1.3824 5.632 2.56 11.8784 2.176 10.5984h198.656l5.8112-26.624 4.2752-20.352 2.7648-4.0448 17.4848-11.4176c66.0736-43.1872 105.8304-115.8656 105.8304-194.56 0-128.6144-105.1648-232.8832-234.9312-232.8832z m-54.3232 49.7408a31.7952 31.7952 0 0 1 37.12 25.2416c3.2 17.152-8.192 33.664-25.4464 36.864-56.96 10.5216-83.0208 44.544-86.784 112.4608a31.744 31.744 0 0 1-33.536 29.7984 31.6672 31.6672 0 0 1-30.0032-33.28c5.376-96.3584 51.584-154.9824 138.6496-171.0848z"
                        p-id="16389"
                        fill="#f4ea2a"></path>
                </svg>
            </div>
            <el-text size="small" style="color: white"
                >温馨提示：{{ tip }}</el-text
            >
        </div>
    </div>
</template>

<style scoped>
.container {
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100vh;
    background-size: cover;
    background-position: center;
}
.centered-div {
    text-align: center;
}
.ms-title {
    padding-top: 5px;
    padding-bottom: 5px;
    padding-left: 20px;
    padding-right: 20px;
    border-radius: 99px;
    background: rgba(255, 255, 255, 0.3);
    overflow: hidden;
}
</style>
