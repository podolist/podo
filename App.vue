<script>
import Vue from 'vue';
export default {
  onLaunch: function() {
    uni.getSystemInfo({
      success: function(e) {
        // #ifndef MP
        Vue.prototype.StatusBar = e.statusBarHeight;
        if (e.platform == 'android') {
          Vue.prototype.CustomBar = e.statusBarHeight + 50;
        } else {
          Vue.prototype.CustomBar = e.statusBarHeight + 45;
        }
        // #endif
      }
    });
  }
};
</script>

<style>
@import 'colorui/main.css';
@import 'colorui/icon.css';
</style>
