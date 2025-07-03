<template>
  <v-container class="fill-height">
    <v-responsive
      class="align-centerfill-height mx-auto"
      max-width="1200"
    >
      <v-img
        class="mb-4"
        height="150"
        src="@/assets/logo.png"
      />

      <div class="text-center">
        <div class="text-h3 font-weight-light mb-4">智感测痛</div>
        <h1 class="text-body-3">基于人工智能集成生命体征检测的自动化痛觉测量系统</h1>
      </div>

      <div class="py-4" />

      <v-row>
        <v-col cols="8" >
          <v-card
            class="py-4"

            :color="ws_status === '已连接' ? 'surface-variant' : 'error'"
            rounded="lg"
            variant="outlined"
            height="300px"
            :title="String('实时力传感器数据 - ') + ws_status"
            prepend-icon="mdi-chart-line"
          >
          <div ref="force_chart" style="width: 100%; height: 100%;"></div>

          </v-card>

        </v-col>
        <v-col cols="4" >
          <v-card
            class="py-4"
            :color="ws_status === '已连接' ? 'surface-variant' : 'error'"

            rounded="lg"
            variant="outlined"
            height="300px"
            :title="String('控制台 - ') + ws_status"
            prepend-icon="mdi-keyboard"
          >
          <v-btn
            v-for="key in keys"
            :key="key"
            :active="activeKey === key"
            @click="sendMessage(key)"
            :id="key"
          >
        {{ key }}
        </v-btn>

        <v-btn
            @click="sendMessage('F')"
            id="trigger"
            :active="activeKey === 'F'"
            color="green"
          >
        触发
        </v-btn>

        <v-btn
            @click="sendMessage('N')"
            id="normal_quit"
            :active="activeKey === 'N'"
            color="blue"
          >
        正常结束
        </v-btn>

        <v-btn
            @click="sendMessage('E')"
            id="error_quit"
            :active="activeKey === 'E'"
            color="red"
          >
        异常结束
        </v-btn>

        <v-btn
            @click="sendMessage('R')"
            id="resume"
            :active="activeKey === 'R'"
            color="yellow"
          >
        恢复
        </v-btn>

          </v-card>

        </v-col>

        <v-col cols="6">
          <v-card

            append-icon=""
            class="py-4"
            color="surface-variant"
            href="/down"
            prepend-icon="mdi-camera-marker"
            rel="noopener noreferrer"
            rounded="lg"
            title="足底传感器"
            variant="text"
            height="550px"
          >
          <img src="http://localhost:8081/down" style="height: 450px;" class="ml-3">
            <v-overlay
              opacity=".06"
              scrim="primary"
              contained
              model-value
              persistent
            />
          </v-card>
        </v-col>

        <v-col cols="6">

          <v-card

            append-icon=""
            class="py-4"
            color="surface-variant"
            href="/top"
            prepend-icon="mdi-camera-lock"
            rel="noopener noreferrer"
            rounded="lg"
            title="行为学传感器"
            variant="text"
            height="550px"
          >
          <img src="http://localhost:8080/top" style="height: 450px;" class="ml-3 center">

            <v-overlay
              opacity=".06"
              scrim="primary"
              contained
              model-value
              persistent
            />
          </v-card>
        </v-col>



      </v-row>
    </v-responsive>
  </v-container>
</template>

<script>
import { ref } from 'vue';
import { tr } from 'vuetify/locale';


export default {
  data() {
    return {
      forceValues: [],
      timeLabels: [],
      myChart: null,
      socket: null,
      keys: ['W', 'S', 'A', 'D'],
      trigger: ['F', 'Q', 'R'],
      activeKey: null,
      ws: null,
      ws_status: '未连接',
      option: {
        tooltip: {},

        xAxis: {
            type: 'category',
            data: this.timeLabels,
        },
        yAxis: {
            type: 'value',
            name: 'Force',
        },
        series: [{
            name: 'Force',
            type: 'line',
            data: this.forceValues,
            smooth: true,
            itemStyle: {
                color: '#5470C6'
            },
            showSymbol: false
        }]
        }
    };
  },
  mounted() {
    this.initChart();
     // 监听键盘事件
    window.addEventListener("keydown", this.handleKeydown);
    window.addEventListener("keyup", this.handleKeyup);
    this.ws = new WebSocket('ws://localhost:9000', 'echo-protocol');
    this.ws.onopen = () => {
            console.log('Connected to server');
            this.ws_status = '已连接';
        };

    this.ws.onmessage = (event) => {
            const li = document.createElement('li');
            console.log(event.data);
            try {
              this.forceValues.push(JSON.parse(event.data));
            }
            catch (e) {
              console.log(e);
            }
            // console.log(this.forceValues);

            if (this.forceValues.length > 400) {
                this.forceValues.shift();
                this.forceValues.pop();
            }

            this.option.series[0].data = this.forceValues;
            this.myChart.setOption(this.option);
        };

    this.ws.onclose = () => {
            console.log('Disconnected from server');
            this.ws_status = '未连接';
        };
  },
  methods: {
    //
    initChart() {

      this.myChart = echarts.init(this.$refs.force_chart);

      this.myChart.setOption(this.option);
    },
    handleKeydown(event) {
      const key = event.key.toUpperCase(); // 转为大写
      if (this.keys.includes(key) || this.trigger.includes(key)) {
        this.activeKey = key; // 激活对应按钮
        this.sendMessage(key); // 发送消息
      }
    },
    // 处理键盘释放事件
    handleKeyup() {
      this.activeKey = null; // 取消激活
    },
    // 发送消息
    sendMessage(key) {
      this.ws.send(key);
      console.log(key);
      // this.socket.send(JSON.stringify({ key }));
    }

  }
};

</script>

<style scoped>

/* 设置wasd的位置 */
#W {
  position: absolute;
  top: 40%;
  left: 50%;
  transform: translate(-50%, -50%);
}

#S {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, 50%);
}

#A {
  position: absolute;
  top: 50%;
  left: 25%;
  transform: translate(-50%, 50%);
}

#D {
  position: absolute;
  top: 50%;
  left: 75%;
  transform: translate(-50%, 50%);
}

#trigger {
  position: absolute;
  top: 70%;
  left: 25%;
  transform: translate(-50%, 50%);
}

#quit {
  position: absolute;
  top: 70%;
  left: 75%;
  transform: translate(-50%, 50%);
}
#resume {
  position: absolute;
  top: 70%;
  left: 50%;
  transform: translate(-50%, 50%);
}
</style>
