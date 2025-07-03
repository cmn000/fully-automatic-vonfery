<template>
  <v-container class="fill-height">
    <v-responsive class="align-centerfill-height mx-auto" max-width="1200">
      <!-- ... 其他头部内容保持不变 ... -->
      <div class="text-center">
        <div class="text-h3 font-weight-light mb-4">智感测痛</div>
        <h1 class="text-body-3 mb-2">基于人工智能集成生命体征检测的自动化痛觉测量系统</h1>
        <h2 class="mb-4">{{date}}</h2>
      </div>
      <v-row>
        <!-- 视频卡片保持不变 -->
        <v-col cols="6">
          <v-card
            class="py-4"
            rounded="lg"
            variant="outlined"
            height="600px"
            title="足底传感器"
            prepend-icon="mdi-chart-line"
          >
            <video id="down-video-player" ref="downVideo" controls class="video-player">
              <source :src="down_video_url" type="video/mp4">
            </video>
          </v-card>
        </v-col>
        <v-col cols="6">
          <v-card
            class="py-4"

            rounded="lg"
            variant="outlined"
            height="600px"
            title="行为学传感器"
            prepend-icon="mdi-chart-line"
          >
            <video id="top-video-player" ref="topVideo" controls class="video-player">
              <source :src="top_video_url" type="video/mp4">
            </video>
          </v-card>
        </v-col>

<!--        &lt;!&ndash; 添加播放按钮 &ndash;&gt;-->
<!--        <v-col cols="12" class="text-center">-->
<!--          <v-btn @click="syncPlay" color="primary" class="mr-2">播放</v-btn>-->
<!--          <v-btn @click="syncPause" color="secondary">暂停</v-btn>-->
<!--        </v-col>-->

        <v-col cols="12" >
          <v-card
            class="py-4"
            rounded="lg"
            variant="outlined"
            height="300px"
            title="力传感器数据"
            prepend-icon="mdi-chart-line"
          >
            <div ref="force_chart" style="width: 100%; height: 100%;"></div>

          </v-card>

        </v-col>

        <!-- 进度条 -->
<!--        <v-col cols="12">-->
<!--          <v-slider-->
<!--            v-model="progress"-->
<!--            :max="1"-->
<!--            :step="0.01"-->
<!--            thumb-label-->

<!--          @change="handleSliderInput"-->
<!--            class="ml-6 mr-6"-->
<!--          ></v-slider>-->
<!--        </v-col>-->
      </v-row>
    </v-responsive>
  </v-container>
</template>

<script>
export default {
  data() {
    return {
      date: null,
      top_video_url: null,
      down_video_url: null,
      progressUpdateInterval: null,
      progress: null,
      isSliding: false, // 标记是否正在拖动滑块
      option: {
        tooltip: {},

        xAxis: {
          type: 'category',
          data: [],
        },
        yAxis: {
          type: 'value',
          name: 'Force',
        },
        series: [{
          name: 'Force',
          type: 'line',
          data: [],
          smooth: true,
          itemStyle: {
            color: '#5470C6'
          },
          showSymbol: false
        }]
      }
    };
  },
  watch: {
    progress(newVal) {
      console.log(this.isSliding);
      if (!this.isSliding) return;
      console.log(newVal);
      this.syncVideoTime(newVal);
    }
  },
  mounted() {
    // 监听视频元数据加载
    const key = this.$route.params.id
    this.date =  this.$route.params.id.slice(0, 4) + "年" + key.slice(4, 6) + "月" + key.slice(6, 8) + "日" + key.slice(8, 10) + ":" + key.slice(10, 12) + ":" + key.slice(12, 14);
    this.top_video_url = this.$server_address + '/videos/' + key + '_top.mp4';
    this.down_video_url = this.$server_address + '/videos/' + key + '_down.mp4';
    this.$refs.topVideo.addEventListener('loadedmetadata', this.initVideos);
    this.$refs.downVideo.addEventListener('loadedmetadata', this.initVideos);
    this.initVideoSync();
    const forceDataUrl = this.$server_address + '/force';
    const formData = new FormData();
    formData.append("id", key);
    fetch(forceDataUrl, {
      method: 'POST',
      body: formData,
    })
      .then(response => response.json())
      .then(data => {
        console.log(data);
        for (let i = 0; i < data.length; i++) {
          // const key = data[i].key
          const value = data[i]['force'];
          this.option.series[0].data.push(value);
          console.log(data[i]['force']);


        }this.initChart();console.log(this.option);
      });

  },
  methods: {
    async initVideoSync() {
      // 等待视频元数据加载
      await Promise.all([
        new Promise(resolve => this.$refs.topVideo.onloadedmetadata = resolve),
        new Promise(resolve => this.$refs.downVideo.onloadedmetadata = resolve)
      ]);

      // 计算播放参数
      const topDuration = this.$refs.topVideo.duration;
      const downDuration = this.$refs.downVideo.duration;
      this.maxDuration = Math.max(topDuration, downDuration);

      // 设置播放速率
      if (topDuration > downDuration) {
        this.$refs.downVideo.playbackRate = downDuration / topDuration;
        this.playbackRate = downDuration / topDuration;
      } else {
        this.$refs.topVideo.playbackRate = topDuration / downDuration;
        this.playbackRate = topDuration / downDuration;
      }
    },

    syncPlay() {
      this.isSyncing = true;

      this.$refs.topVideo.play();
      this.$refs.downVideo.play();
      this.updateProgress();
    },

    syncPause() {
      this.isSyncing = false;
      cancelAnimationFrame(this.animationFrame);
      this.$refs.topVideo.pause();
      this.$refs.downVideo.pause();
    },

    handleSeek(time) {
      this.currentTime = time;
      this.$refs.topVideo.currentTime = Math.min(
        time * (this.$refs.topVideo.duration / this.maxDuration),
        this.$refs.topVideo.duration
      );
      this.$refs.downVideo.currentTime = Math.min(
        time * (this.$refs.downVideo.duration / this.maxDuration),
        this.$refs.downVideo.duration
      );
    },

    updateProgress() {
      if (!this.isSyncing) return;

      // 以较长视频的时间为基准
      this.currentTime = Math.min(
        Math.max(
          this.$refs.topVideo.currentTime / (this.$refs.topVideo.duration / this.maxDuration),
          this.$refs.downVideo.currentTime / (this.$refs.downVideo.duration / this.maxDuration)
        ),
        this.maxDuration
      );

      this.animationFrame = requestAnimationFrame(this.updateProgress);
    },

    formatTime(seconds) {
      const date = new Date(0);
      date.setSeconds(seconds);
      return date.toISOString().substr(11, 8);
    },

    // 处理原生控件操作
    handleVideoPlay() {
      if (!this.isSyncing) this.syncPlay();
    },

    handleVideoPause() {
      if (this.isSyncing) this.syncPause();
    },
    initChart() {

      this.myChart = echarts.init(this.$refs.force_chart);

      this.myChart.setOption(this.option);
    },
  }
};
</script>


<style scoped>



body {
  font-family: Arial, sans-serif;
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
}
#video-container {
  margin-bottom: 20px;
}
.video-player {
  width: 100%;
}
#progress-info {
  margin-top: 10px;
  padding: 10px;
  background-color: #f0f0f0;
  border-radius: 4px;
}
#video-list {
  margin-top: 20px;
}
.video-item {
  padding: 8px;
  margin-bottom: 5px;
  background-color: #f8f8f8;
  cursor: pointer;
}
.video-item:hover {
  background-color: #e8e8e8;
}
#upload-section {
  margin-top: 20px;
  padding: 15px;
  background-color: #f5f5f5;
  border-radius: 4px;
}
</style>
