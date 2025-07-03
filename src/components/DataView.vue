<template>
  <v-container class="fill-height">
    <v-responsive
      class="align-centerfill-height mx-auto"
      max-width="1200"
    >
    <div class="text-center">
        <div class="text-h3 font-weight-light mb-4">智感测痛</div>
        <h1 class="text-body-3 mb-4">基于人工智能集成生命体征检测的自动化痛觉测量系统</h1>
      </div>

      <!-- 创建一个列表 -->
      <v-row>
        <v-col cols="12">
      <v-list>
       <v-list-item v-for="item in items" :key="item.title" class="mb-4">
        <v-list-item-title>{{ item.title }}</v-list-item-title>
        <v-list-item-subtitle>{{ item.subtitle }} <v-btn class="right-0" :href="'/detail/' + item.id">查看详情</v-btn></v-list-item-subtitle>
      </v-list-item>
      </v-list>

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
      items: [

      ],
    };
  },
  mounted() {
    this.getAllData();
  },
  methods: {
      getAllData() {
        const url = this.$server_address + "/data";
        var recv = null;
        fetch(url)
          .then(response => response.json())
          .then(data => {
            recv = data;
            console.log(recv);
            // 获取每一个键值对的键
            for (let key in recv) {
              console.log(key);
              // 按照YYYYMMDDHHMMSS的格式获取时间
              let time = key.slice(0, 4) + "年" + key.slice(4, 6) + "月" + key.slice(6, 8) + "日" + key.slice(8, 10) + ":" + key.slice(10, 12) + ":" + key.slice(12, 14);
              const item = { title: time, subtitle: recv[key].info + " " + recv[key].force, id: key};
              this.items.push(item);
            }
          });
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

body {
            font-family: Arial, sans-serif;
            max-width: 800px;
            margin: 0 auto;
            padding: 20px;
        }
        #video-container {
            margin-bottom: 20px;
        }
        #video-player {
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
