<template>
  <div class="qun-player"
       ref="container"
       @click.stop="isClearMode=!isClearMode">
    <!--模拟poster -->
    <div class="_poster"
         :style="{backgroundImage:`url(${options.cover})`}"
         v-show="!isPlaying&&isStart">
    </div>
    <template v-show="isPlaying">
      <video class="_video-ref"
             webkit-playsinline
             playsinline
             x5-video-player-fullscreen
             x-webkit-airplay="allow"
             x5-video-player-type="h5"
             crossorigin="anonymous"
             ref="video"
             :muted="options.muted"
             :loop="options.loop"
             :preload="options.preload"
             :poster="options.cover">
        <source v-for="(item, index) in vUrl"
                :key="index"
                :src="item"
                :type="`video/${getUrlType(item)}`">
        Your browser does not support the video element.
      </video>
      <transition name="fade">
        <PlayBtn :isPlaying.sync="isPlaying"
                 v-show="!isClearMode" />
      </transition>
      <transition name="fade">
        <BaseControls @paused="handlePaused"
                      @fullscreen="$emit('fullscreen',$event)"
                      v-show="!isClearMode" />
      </transition>
    </template>
  </div>
</template>
<script>
import BaseControls from './BaseControls';
import PlayBtn from './PlayBtn';
const VERSION = require('../../../package.json').version;

export default {
  name: 'VueMiniPlayer',
  components: {
    BaseControls,
    PlayBtn
  },
  props: {
    video: {
      type: Object,
      default: function() {
        return {};
      }
    },
    mutex: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      baseVideo: {
        url: '',
        cover: '',
        muted: true,
        loop: false,
        preload: 'metadata',
        poster: '',
        volume: 1,
        autoplay: false
      },
      $video: null,
      $container: null,
      clearModeTimer: null,
      isStart: true,
      isPlaying: false,
      isClearMode: false
    };
  },
  watch: {
    isPlaying() {
      this.isStart = false;
      this.play();
      this.setClearModeTimer();
    }
  },
  computed: {
    vUrl() {
      let url = this.video.url || [];
      if (typeof url === 'string') {
        url = [url];
      } else if (Object.prototype.toString.call(url) === '[object Object]') {
        console.warn(new Error('视频URL只接受String或者Array'));
        return [];
      }
      return url;
    },
    // 合并默认和用户自定义属性配置
    options() {
      return Object.assign({}, this.baseVideo, this.video);
    }
  },
  methods: {
    getUrlType(url) {
      let u = url.split('?')[0] + '?v=1';
      return u.match(/[^\.]+(?=\?)/) || 'mp4';
    },
    init() {
      this.$video = this.$refs.video;
      this.$container = this.$refs.container;
      this.isPlaying = true;
      this.$video.load();
      this.initPlayer();
      this.$emit('ready');
      setTimeout(() => {
        this.isPlaying = this.options.autoplay;
      }, 300);
    },
    initPlayer() {
      this.$video.volume = this.options.volume;
      setTimeout(() => {
        this.$video.muted = false;
      }, 500);
    },
    setClearModeTimer() {
      if (this.clearModeTimer) {
        clearTimeout(this.clearModeTimer);
      }
      this.clearModeTimer = setTimeout(() => {
        this.isClearMode = true;
        this.$emit('clearMode');
      }, 3000);
    },
    pauseAllVideo() {
      if (this.mutex) {
        const videos = document.querySelectorAll('video');
        videos.forEach(v => {
          v.pause && v.pause();
        });
      }
    },
    play() {
      if (this.isPlaying) {
        this.pauseAllVideo();
        this.$video.play();
      } else {
        this.$video.pause();
      }
      this.$emit('videoPlay', this.isPlaying);
    },
    handlePaused() {
      this.isPlaying = false;
    }
  },
  created() {
    this.$emit('created');
    this.$nextTick(() => {
      this.init();
    });
  },

  mounted() {
    this.$emit('mounted');
    console.log(
      '\n' + ' %c vue-mini-player v' + VERSION + ' %c https://github.com/webweifeng/vue-mini-player ' + '\n' + '\n',
      'color: #fadfa3; background: #030307; padding:5px 0;',
      'background: #fadfa3; padding:5px 0;'
    );
  },
  updated() {},
  beforeDestroy() {
    this.$emit('beforeDestroy');
  },
  destroyed() {
    this.$emit('destroyed');
  }
};
</script>
<style lang="scss" scoped>
.qun-player {
  width: 100%;
  height: 100%;
  min-height: 10em;
  position: relative;
  background: #000;
  overflow: hidden;
  &:fullscreen,
  &:-webkit-full-screen,
  &:-moz-full-screen,
  &:-ms-fullscreen {
    width: 100%;
    height: 100%;
    position: fixed;
    z-index: 100000;
    left: 0;
    top: 0;
    margin: 0;
    padding: 0;
    transform: translate(0, 0);
  }
  ._poster {
    width: 100%;
    height: 100%;
    background-size: cover;
    background-position: center;
    background-repeat: no-repeat;
  }
  ._video-ref {
    background: #000;
    width: 100%;
    height: 100%;
    /* object-fit: cover; */
    &::-webkit-media-controls,
    &::-webkit-media-controls-enclosure {
      display: none !important;
    }
  }
}
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s;
}
.fade-enter, .fade-leave-to /* .fade-leave-active below version 2.1.8 */ {
  opacity: 0;
}
</style>

