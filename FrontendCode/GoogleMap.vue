<template>
  <div className="map-container" ref="mapContainer"></div>
</template>

<script>
/* global google */
export default {
  name: "GoogleMap",
  mounted() {
    if (window.google) {
      this.initMap();
    } else {
      console.error("Google Maps API 未加载！");
    }
  },
  methods: {
    initMap() {
      // 修改后的UIC campus 坐标
      const centerCoords = { lat: 41.8708, lng: -87.6505 };
      new google.maps.Map(this.$refs.mapContainer, {
        center: centerCoords,
        zoom: 12,
      });
    },
  },
};
</script>

<style scoped>
.map-container {
  width: 100%;
  height: 200px;
}
</style>
