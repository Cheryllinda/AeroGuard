<template>
  <div class="home-container">
    <!-- 半透明覆盖层：让背景图呈现 10% 透明效果 -->
    <div class="background-overlay"></div>

    <!-- 顶部导航/标题栏 -->
    <header class="top-bar">
      <div class="left-title">
        <img class="logo" src="@/assets/logo.svg" alt="Logo" />
        <!-- 根据当前语言显示标题 -->
        <h2>{{ t('title') }}</h2>
      </div>

      <div class="right-info">
        <!-- 显示当前时间 -->
        <span>{{ currentDate }}</span>
        <!-- 语言切换按钮 -->
        <div class="lang-switch">
          <button @click="setLanguage('en')">EN</button>
          <button @click="setLanguage('cn')">中文</button>
        </div>
      </div>
    </header>

    <!-- 主体布局：左侧 + 中间 + 右侧 -->
    <div class="main-content">
      <!-- 左侧区：地图、统计数据等 -->
      <aside class="left-panel">
        <div class="map-container">
          <!-- 将原来的静态地图图像替换为 GoogleMap 组件 -->
          <GoogleMap />
        </div>
        <div class="stats-container">
          <h3>{{ t('siteOverview') }}</h3>
          <ul>
            <li>
              <img src="@/assets/one-people-on-site.svg" class="stat-icon" />
              {{ t('peopleOnSite') }}: {{ siteStats.people }}
            </li>
            <li>
              <img src="@/assets/one-vehicle-on-site.svg" class="stat-icon" />
              {{ t('vehicleOnSite') }}: {{ siteStats.vehicles }}
            </li>
            <li>
              <img src="@/assets/a-construction-helmet.svg" class="stat-icon" />
              {{ t('helmetCheck') }}: {{ siteStats.helmet_percent }}%
            </li>
            <li>
              <img src="@/assets/fire-icon.svg" class="stat-icon" />
              {{ t('fireRisk') }}:
              <span :style="{color: siteStats.fire_risk ? 'red':'green'}">
                {{ siteStats.fire_risk ? t('yes') : t('no') }}
              </span>
            </li>
          </ul>
        </div>
      </aside>

      <!-- 中间区：视频画面等 -->
      <section class="center-panel">
        <!-- 第一个视频模块 -->
        <div class="video-box">
          <h3>{{ t('uavLive') }}</h3>
          <!-- 使用 hls.js 播放 HLS 流，播放地址取自 uavData.hls -->
          <div class="video-wrapper">
            <video id="hls-video" controls>
              {{ t('noVideoSupport') }}
            </video>
          </div>
        </div>

        <!-- 第二个视频模块：可动态加载 /videos 下的 MP4 文件 -->
        <div class="video-box">
          <h3>{{ t('constructionCam') }}</h3>
          <div class="video-wrapper">
            <!-- 注意这里使用 v-bind(:) 来动态设置 src -->
            <video controls :src="selectedVideo ? ('/videos/' + selectedVideo) : ''">
              {{ t('noVideoSupport') }}
            </video>
          </div>
        </div>

        <!-- 新增的 YOLO 实时检测视频模块 -->
        <div class="video-box">
          <h3>{{ t('realtimeScreenshot') }}</h3>
          <div class="video-wrapper">
            <!-- 使用 img 标签实时展示 MJPEG 流 -->
            <img src="http://154.201.89.38:6700/video_feed" alt="YOLO实时检测" />
          </div>
        </div>
      </section>

      <!-- 右侧区：数据列表、告警信息等 -->
      <aside class="right-panel">
        <div class="table-section">
          <h3>{{ t('latestUAVStatus') }}</h3>
          <table>
            <thead>
              <tr>
                <th>ID</th>
                <th>PlanId</th>
                <th>RecordId</th>
                <th>FlightTime</th>
                <th>RtmpSubUrl</th>
<!--                <th>hlsUrl</th>-->
              </tr>
            </thead>
            <tbody>
              <tr v-for="(item, index) in uavList" :key="index">
                <td>{{ item.id }}</td>
                <td>{{ item.PlanId }}</td>
                <td>{{ item.uavRecordId }}</td>
                <td>{{ item.FlightTime }}</td>
                <td>{{ item.RtmpSubUrl }}</td>
<!--                <td>{{ item.hlsUrl }}</td>-->
              </tr>
            </tbody>
          </table>
        </div>

        <!-- 在右侧新增一个视频列表 -->
        <div class="table-section">
          <h3>Video List</h3>
          <ul style="list-style: none; padding: 0;">
            <li
              v-for="(videoFile, idx) in videoList"
              :key="idx"
              @click="selectVideo(videoFile)"
              style="cursor: pointer; margin: 5px 0; color: blue; text-decoration: underline;"
            >
              {{ videoFile }}
            </li>
          </ul>
        </div>
      </aside>
    </div>
  </div>
</template>

<script>
import GoogleMap from '@/components/GoogleMap.vue';
import Hls from 'hls.js';

export default {
  name: "HomeView",
  components: {
    GoogleMap,
  },
  data() {
    return {
      // UAV 数据
      uavList: [],
      uavData: null,

      // 现场统计
      siteStats: {people:0, vehicles:0, helmet_percent:0, fire_risk:false},
      randomPeopleCount: 82,
      randomVehicleCount: 5,
      randomHelmetPercent: 90,

      // 时间显示
      currentDate: new Date().toLocaleString(),

      // 当前语言
      currentLang: 'en',

      // 简易 i18n 字典
      i18n: {
        en: {
          title: "Construction Site Command and Monitoring Center",
          siteOverview: "Site Overview",
          peopleOnSite: "People on site",
          vehicleOnSite: "Vehicles on site",
          helmetCheck: "Helmet check",
          fireRisk:"Fire risk",
          yes:"YES",
          no:"NO",
          uavLive: "UAV Live Stream",
          constructionCam: "Construction Site Cam",
          noVideoSupport: "Your browser does not support the video tag.",
          realtimeScreenshot: "Real-time detection screenshot",
          noStream: "No stream available",
          latestUAVStatus: "Latest UAVStatus Records"
        },
        cn: {
          title: "某施工场地指挥监控中心",
          siteOverview: "现场监控概览",
          peopleOnSite: "在场人员",
          vehicleOnSite: "工程车辆",
          helmetCheck: "安全帽检测",
          fireRisk:"火灾风险",
          yes:"有",
          no:"无",
          uavLive: "无人机实时画面",
          constructionCam: "施工现场摄像头",
          noVideoSupport: "您的浏览器不支持 video 标签。",
          realtimeScreenshot: "实时检测截图",
          noStream: "暂无视频流",
          latestUAVStatus: "最新 UAVStatus 记录"
        }
      }
    };
  },
  created() {
    this.fetchUavStatus();
    this.fetchVideoList();
    this.pollStats();                         // 首次拉取
    setInterval(this.pollStats, 1000);        // 每秒刷新一次，可自行调整
    // 每隔1秒更新顶部时间
    setInterval(() => {
      this.currentDate = new Date().toLocaleString();
    }, 1000);
  },
  watch: {
    uavData(newVal) {
      // 当 uavData 更新且包含 hlsUrl 属性时初始化播放器
      if (newVal && newVal.hlsUrl) {
        const video = document.getElementById('hls-video');
        const hlsUrl = newVal.hlsUrl;
        if (Hls.isSupported()) {
          const hls = new Hls();
          hls.loadSource(hlsUrl);
          hls.attachMedia(video);
          hls.on(Hls.Events.MANIFEST_PARSED, () => {
            video.play();
          });
        } else if (video.canPlayType('application/vnd.apple.mpegurl')) {
          video.src = hlsUrl;
          video.addEventListener('loadedmetadata', () => {
            video.play();
          });
        }
      }
    }
  },
  methods: {
    // 获取 UAVStatus
    // async fetchUavStatus() {
    //   try {
    //     const res = await fetch("http://localhost:5000/api/uavstatus");
    //     const jsonData = await res.json();
    //     if (jsonData.success) {
    //       this.uavList = jsonData.data;
    //       if (this.uavList.length > 0) {
    //         // 假设 uavList 第一项包含 hls 字段
    //         this.uavData = this.uavList[0];
    //       }
    //     } else {
    //       console.error("API error:", jsonData.error);
    //     }
    //   } catch (err) {
    //     console.error("fetchUavStatus error:", err);
    //   }
    // },
    // 自动获取实时states
    async pollStats() {
    try {
      const res = await fetch("http://154.201.89.38:6700/api/site_stats");
      const json = await res.json();
      if (json.success) this.siteStats = json.data;
    } catch(e){ console.error("pollStats error", e); }
  },
    async fetchUavStatus() {
      try {
        // const res = await fetch(`${process.env.VUE_APP_API_URL}/uavstatus`)
        const res = await fetch("http://154.201.89.38:6700/api/uavstatus")
        const jsonData = await res.json();
        if (jsonData.success) {
          this.uavList = jsonData.data;
          if (this.uavList.length > 0) {
            // 假设 uavList 第一项包含 hls 字段
            this.uavData = this.uavList[0];
          }
        } else {
          console.error("API error:", jsonData.error);
        }
      } catch (err) {
        console.error("fetchUavStatus error:", err);
      }
    },
    // 新增：自动获取视频列表
    async fetchVideoList() {
      try {
        const res = await fetch("http://154.201.89.38:6700/api/videos"); // 修改成你的服务器地址和端口
        const jsonData = await res.json();
        if (jsonData.success) {
          this.videoList = jsonData.files;
          // 默认选中第一个视频（如果存在）
          if (this.videoList.length > 0) {
            this.selectedVideo = this.videoList[0];
          }
        } else {
          console.error("API error:", jsonData.error);
        }
      } catch (err) {
        console.error("fetchVideoList error:", err);
      }
    },
    // 更新当前选择的视频文件
    selectVideo(videoFile) {
      this.selectedVideo = videoFile;
    },
    // 切换语言
    setLanguage(lang) {
      this.currentLang = lang;
    },
    // 读取当前语言下的文案
    t(key) {
      return this.i18n[this.currentLang][key] || key;
    }
  },
};
</script>

<style scoped>
/* 外层容器，使用背景图 */
.home-container {
  position: relative;
  display: flex;
  flex-direction: column;
  height: 100vh;
  background: url("@/assets/bg.png") no-repeat center center fixed;
  background-size: cover;
  font-family: "Microsoft YaHei", sans-serif;
}

/* 半透明覆盖层，让背景图呈现 10% 透明度效果 */
.background-overlay {
  content: "";
  position: absolute;
  top: 0; left: 0; right: 0; bottom: 0;
  background-color: rgba(255, 255, 255, 0.1);
  pointer-events: none;
  z-index: 0;
}

/* 顶部条 */
.top-bar {
  position: relative;
  z-index: 1;
  display: flex;
  align-items: center;
  justify-content: space-between;
  background-color: rgba(44, 62, 80, 0.9);
  color: #fff;
  padding: 10px 20px;
}
.top-bar .left-title {
  display: flex;
  align-items: center;
}
.top-bar .logo {
  width: 40px;
  height: 40px;
  margin-right: 8px;
}
.top-bar h2 {
  margin: 0;
  font-size: 16px;
}
.top-bar .right-info {
  font-size: 14px;
  opacity: 0.9;
  display: flex;
  align-items: center;
}
.lang-switch {
  margin-left: 20px;
}
.lang-switch button {
  margin: 0 5px;
  padding: 4px 8px;
  border: none;
  background: #34495e;
  color: #fff;
  cursor: pointer;
  border-radius: 3px;
}
.lang-switch button:hover {
  background: #3b5998;
}

/* 主体布局：左右三列 */
.main-content {
  position: relative;
  z-index: 1;
  flex: 1;
  display: flex;
  overflow: hidden;
}

/* 左侧区 */
.left-panel {
  flex: 0 0 20%;
  background: rgba(255, 255, 255, 0.8);
  border-right: 1px solid #ddd;
  display: flex;
  flex-direction: column;
  padding: 10px;
  overflow-y: auto;
}
.map-container {
  width: 100%;
  height: 200px;
  overflow: hidden;
  border: 1px solid #ccc;
  margin-bottom: 10px;
}

.stats-container {
  /* 这里可以统一调整容器内文字大小 */
  font-size: 18px;
  /* 如果需要让标题也左对齐且紧贴左边，可以去掉默认的内外边距 */
  /* margin: 0; */
  /* padding: 0; */
}

.stats-container h3 {
  /* 去除默认外边距，方便左对齐 */
  margin: 0 0 10px 0;
  /* 让标题稍微大一点 */
  font-size: 18px;
}

.stats-container ul {
  /* 去除列表默认的缩进和符号 */
  list-style: none;
  margin: 0;
  padding: 0;
}

.stats-container ul li {
  /* 让图标与文字在同一行对齐 */
  display: flex;
  align-items: center;

  /* 适当设置行高或上下间距 */
  margin: 6px 0;

  /* 如果还需要再放大文字，可以在这里调节 */
  /* font-size: 1.2em; */
}

.stat-icon {
  /* 控制图标大小 */
  width: 50px;
  height: 50px;
  /* 图标与文字之间的间距 */
  margin-right: 8px;
}

/* 中间区 */
.center-panel {
  flex: 1;
  display: flex;
  flex-direction: column;
  padding: 10px;
  overflow-y: auto;
  background: rgba(255, 255, 255, 0.6);
}
.video-box {
  background: #fff;
  border: 1px solid #ddd;
  margin-bottom: 10px;
  padding: 10px;
}
.video-box h3 {
  margin-top: 0;
}
.video-wrapper {
  width: 100%;
  background: #000;
  margin-top: 8px;
  padding: 5px;
}
video {
  width: 100%;
  outline: none;
}
.video-wrapper img {
  width: 100%;
  height: auto;
  display: block; /* 避免下方出现空隙 */
  object-fit: contain; /* 根据情况可以选择 contain 或 cover */
}


/* 右侧区 */
.right-panel {
  flex: 0 0 25%;
  background: rgba(255, 255, 255, 0.8);
  border-left: 1px solid #ddd;
  padding: 10px;
  overflow-y: auto;
}
.table-section {
  background: #fafafa;
  padding: 10px;
  border: 1px solid #ddd;
}
.table-section h3 {
  margin-top: 0;
}
.table-section table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 8px;
}
.table-section th,
.table-section td {
  border: 1px solid #ccc;
  padding: 6px 8px;
  font-size: 12px;
  text-align: center;
}
.table-section th {
  background: #f0f0f0;
}
</style>
