<template>
  <view class="hub">
    <cu-custom bgColor="bg-black"></cu-custom>

    <video class="video" v-if="src" :src="src" autoplay></video>

    <!-- 列表 -->
    <view class="cu-modal bottom-modal" :class="{ show: modal === 'list' }">
      <view class="cu-dialog">
        <view class="cu-bar bg-white">
          <view class="action text-green"></view>
          <view class="action text-blue" @tap="modal = null">关闭</view>
        </view>
        <view class="padding-sm">
          <scroll-view scroll-y style="height: 70vh;">
            <view class="cu-list menu">
              <view class="cu-item" v-for="(item, inex) in downloadingList" :key="inex" @longpress="onWillDelete(item)">
                <view class="content">
                  <text class="text-grey">{{ item.title }} - {{ item.progress }}%</text>
                </view>
                <view class="action">
                  <text class="text-blue text-sm">{{ item.size }}</text>
                </view>
              </view>
              <view class="cu-item" v-for="(item, inex) in list" @click.stop="onPlay(item)" @longpress="onWillDelete(item)" :key="inex">
                <view class="content">
                  <text class="text-grey">{{ item.title }}</text>
                </view>
                <view class="action"><text class="cuIcon-playfill text-blue"></text></view>
              </view>
            </view>
          </scroll-view>
        </view>
      </view>
    </view>

    <!-- 确认删除 -->
    <view class="cu-modal" v-if="deleting" :class="deleting ? 'show' : ''">
      <view class="cu-dialog">
        <view class="cu-bar bg-white justify-end">
          <view class="content">确认删除?</view>
          <view class="action" @tap="deleting = null"><text class="cuIcon-close text-red"></text></view>
        </view>
        <view class="padding-xl">{{ deleting.title }}</view>
        <view class="cu-bar bg-white justify-end">
          <view class="action">
            <button class="cu-btn line-green text-green" @tap="deleting = null">取消</button>
            <button class="cu-btn bg-green margin-left" @tap="onDelete">确定</button>
          </view>
        </view>
      </view>
    </view>

    <!-- 清晰度 -->
    <view class="cu-modal bottom-modal" :class="{ show: modal === 'download' }">
      <view class="cu-dialog">
        <view class="cu-bar bg-white">
          <view class="action text-green" v-if="currentInfo">{{currentInfo.title}}</view>
          <view class="action text-blue" @tap="modal = null">关闭</view>
        </view>
        <view class="padding-sm">
          <scroll-view scroll-y style="height: 50vh;" v-if="currentInfo">
            <view class="cu-list menu">
              <view class="cu-item" v-for="(item, inex) in currentInfo.list" @click.stop="download(item)" :key="inex">
                <view class="content">
                  <text class="text-grey">{{ item[0] }}</text>
                </view>
              </view>
            </view>
          </scroll-view>
        </view>
      </view>
    </view>

    <view class="cu-bar foot input">
      <button :class="{ hide: src }" class="cu-btn bg-black round shadow" @click="goback()"><text class="cuIcon-back text-white"></text></button>
      <button :class="{ hide: src }" class="cu-btn bg-black round shadow" @click="refresh()"><text class="cuIcon-refresh text-white"></text></button>
      <button class="cu-btn bg-gradual-blue  round shadow" @click="onEscPlay"><text class="cuIcon-list text-white"></text></button>
      <button :class="{ hide: src || !currentInfo }" class="cu-btn bg-black round shadow" @click="modal = 'download'"><text class="cuIcon-down text-white"></text></button>
      <button :class="{ hide: !src }" class="cu-btn bg-red round shadow" @click="onEscPlay"><text class="cuIcon-close text-white"></text></button>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      downloadingList: [],
      list: [],
      modal: null,
      deleting: null,
      src: null,
      wv: null,
      currentInfo: null
    };
  },
  onLoad({ url = 'https://baidu.com' }) {
    // #ifdef APP-PLUS

    let wv = plus.webview.create('', '', {
      top: uni.getSystemInfoSync().statusBarHeight,
      bottom: 50
    });

    wv.addEventListener('titleUpdate', this.onPostMessage);

    wv.appendJsFile('/static/info.js');

    wv.loadURL(decodeURIComponent(url));

    let currentWebview = this.$mp.page.$getAppWebview();

    currentWebview.append(wv);

    this.wv = wv;
    // #endif
  },
  onShow() {
    this.list = uni.getStorageSync('v') || [];
  },
  watch: {
    list(list) {
      uni.setStorageSync('v', list);
    },
    modal(type) {
      if (!type && !this.src) {
        this.wv.show();
      } else {
        this.wv.hide();
      }
    }
  },
  methods: {
    goback() {
      this.wv.back();
    },
    refresh() {
      this.wv.reload();
    },
    onPostMessage(event) {
      let dataStr = event.title;

      try {
        let data = JSON.parse(dataStr);
        let { title, ...rest } = data;

        if (!title || !Object.keys(rest).length) {
          throw Error();
        }

        this.currentInfo = {
          title,
          list: Object.entries(rest)
        };
      } catch (error) {
        this.currentInfo = null;
      }
    },
    onEscPlay() {
      this.src = null;
      this.modal = 'list';
    },
    onWillDelete(item) {
      this.deleting = item;
    },
    onDelete() {
      let task = this.deleting.task;
      let src = this.deleting.src;

      if (task) {
        task.abort();

        this.downloadingList.splice(this.downloadingList.findIndex(item => src === item.src), 1);
      } else {
        uni.removeSavedFile({
          filePath: src
        });

        this.list.splice(this.list.findIndex(item => src === item.src), 1);
      }

      this.deleting = null;
    },
    onPlay(item) {
      this.src = item.src;
      this.modal = null;
    },
    download([_title, src]) {
      if (this.downloadingList.some(item => item.src === src)) {
        return;
      }
      
      this.model = 'list';
      
      let title = this.currentInfo.title;

      let index = this.downloadingList.length;

      let downloadTask = uni.downloadFile({
        url: src,
        success: res => {
          if (res.statusCode === 200) {
            uni.saveFile({
              tempFilePath: res.tempFilePath,
              success: res => {
                this.list.push({
                  title,
                  src: res.savedFilePath
                });
              }
            });
          } else {
            let item = this.downloadingList[index];
            
            item.title = '出现错误，请长按删除';
            this.$set(this.downloadingList, index, item);
            
            downloadTask.abort();
          }
        },
        fail: () => {
          let item = this.downloadingList[index];
          
          item.title = '出现错误，请长按删除';
          this.$set(this.downloadingList, index, item);

          downloadTask.abort();
        }
      });

      downloadTask.onProgressUpdate(res => {
        let item = this.downloadingList[index];

        let size = `${Number(res.totalBytesWritten / 1000000).toFixed(2)}/${Number(res.totalBytesExpectedToWrite / 1000000).toFixed(2)} mb`;

        item.progress = res.progress;
        item.size = size;

        if (+res.progress === 100) {
          this.downloadingList.splice(index, 1);
        } else {
          this.$set(this.downloadingList, index, item);
        }
      });

      this.downloadingList.push({
        title,
        src,
        progress: 0,
        size: '',
        task: downloadTask
      });
      
      this.modal = 'list';
    }
  }
};
</script>

<style>
.video {
  margin-top: 100px;
  width: 100vw;
}

.foot {
  padding: 0 12px;
}

.cu-btn.hide {
  opacity: 0;
  height: 0;
  pointer-events: none;
}
</style>
