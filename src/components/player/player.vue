<template>
  <div id="player">
    <play @show="Show"
          :PlayToggle="PlayToggle"
          @Pause="Pause"
          @PlayInfoShow="PlayInfoShow"
    ></play>
    <div class="player-list" v-if="lists.length">
      <div id="player-title">
        <i></i>
      </div>
      <div class="scroll-div" ref="scrollDiv" @click="Close">
        <div class="scroll-div2" ref="scrollDiv2">
          <div class="scroll-music-title">
            <div class="smt_left">
              <div @click.stop="Mode">
                <span ref="IconMode"><i :class="iconMode"></i></span>
                <span>{{playModeName[playModeNum % 3]}}</span>
              </div>
            </div>
            <div><i class="icon iconfont icon-xiazai"></i></div>
            <div><i class="icon iconfont icon-tianjia"></i></div>
            <div><i class="icon iconfont icon-lajixiang"></i></div>
          </div>
          <div class="scroll-list">
            <scroll ref="scroll">
              <Div class="music">
                <ul>
                  <li v-for="list,index in lists"
                      :class="{'active': index === currentIndex}"
                      @click="PlayMusic(index,list.id)"
                  >
                    <span>{{list.songname}}<small> - {{list.singer}}</small></span>
                    <i class="icon iconfont icon-favorite-outline"></i>
                    <i class="icon iconfont icon-add"></i>
                  </li>
                </ul>
              </Div>
            </scroll>
          </div>
          <div class="close" @click="Close">
            <span>关闭</span>
          </div>
        </div>
      </div>
    </div>
    <transition name="fade">
      <div id="play-music-info" v-if="lists.length" v-show="PlayMusicInfoShow">
          <div class="play-music-bg" :style="{'backgroundImage':`url(${lists[currentIndex].songImage})`}"></div>
          <div class="content">
            <div class="play-music-info-top">
              <i class="icon iconfont icon-xiala2" @click="PlayInfoHide"></i>
              <div class="play-music-info-title">
                <span>{{lists[currentIndex].songname}}</span>
              </div>
              <i class="icon iconfont icon-listmore"></i>
            </div>
            <div class="play-music-info-center"
                 @touchstart="TouchStart"
                 @touchmove="TouchMove"
                 @touchend = "TouchEnd"
            >
              <transition name="content">
                <div v-show="show % 3 === 0" class="play-music-info-content">
                  <ul>
                    <li>
                      <div><img src=""/></div>
                      <div><p>歌手：赵雷</p></div>
                    </li>
                    <li>
                      <div><img src=""/></div>
                      <div><p>专辑：十九岁</p></div>
                    </li>
                  </ul>
                </div>
              </transition>
              <transition name="image">
                <div v-show="show % 3 === 1" class="play-music-info-image">
                  <p>— {{lists[currentIndex].singer}} —</p>
                  <p>
                    <span>标准</span>
                    <span>音效</span>
                  </p>
                  <div class="song-image" :class="{'playing': !playing}">
                    <img :src="lists[currentIndex].songImage"/>
                  </div>
                  <div>
                    <p>歌词</p>
                  </div>
                  <ul class="button-show">
                    <li :class="{'active':show % 3 === 0 }"></li>
                    <li :class="{'active':show % 3 === 1 }"></li>
                    <li :class="{'active':show % 3 === 2 }"></li>
                  </ul>
                </div>
              </transition>
              <transition name="lyric">
                <div v-show="show % 3 === 2" class="play-music-info-lyric"></div>
              </transition>
            </div>
            <div  class="play-music-info-bottom">
              <div class="play-music-info-time">
                <span class="time-start">{{format(currentTime)}}</span>
                <div class="process" ref="processDiv">
                  <div class="process-bar2" ref="progress"></div>
                  <div class="process-bar-btn"  ref="progressBtn"></div>
                </div>
                <span class="time-end">{{format(lists[currentIndex].interval)}}</span>
              </div>
              <div class="play-music-info-handler">
                <i @click.stop="Mode" class="icon iconfont text-r" :class="iconMode"></i>
                <i class="icon iconfont icon-shangyishou text-r"></i>
                <i class="icon iconfont" :class="PlayToggle" @click="Pause"></i>
                <i class="icon iconfont icon-xiayishou text-l"></i>
                <i class="icon iconfont icon-menu text-l"></i>
              </div>
              <div class="play-music-info-button">
              </div>
            </div>
          </div>
        </div>
    </transition>
    <audio v-if="lists.length"
           ref="Audio"
           :src="lists[currentIndex].mp3Url"
           @timeupdate="timeupdate"
           autoplay></audio>
  </div>
</template>

<script type="es6">
import Play from '@/base/play/play'
import Scroll from '@/base/scroll/scroll.vue'
import { mapGetters, mapActions, mapMutations } from 'vuex'

const progressBtnWidth = 16 //播放按钮圆点的宽度
export default {
  components: {Scroll, Play},
  data () {
    return {
      playModeName: ['单曲循环', '顺序播放', '随机播放'],
      show: 1,  //滑动 0 1 2 显示3个页面
      currentTime: 0,
      PlayMusicInfoShow: false
    }
  },
  created () {
    this.touches = {} //用来保存滑动的变量
  },
  computed: {
      ...mapGetters({
        lists: 'PlayMusicList',
        currentIndex: 'currentIndex',
        playing: 'playing',
        PLAY_MODE: 'playMode',
        playModeNum: 'playModeNum'
      }),
      iconMode () {
        return this.playModeNum % 3 === this.PLAY_MODE.sequence ? 'icon iconfont icon-xunhuanbofang'
          : this.playModeNum % 3 === this.PLAY_MODE.random ? 'icon iconfont icon-random'
            : 'icon iconfont icon-danquxunhuan'
      },
      PlayToggle () {
        return this.playing ? 'icon-zanting' : 'icon-play-circle'
      },
      processWidth () {
        if (this.lists.length) return this.currentTime / this.lists[this.currentIndex].interval
      }
    },
  methods: {
    ...mapActions({
      PLAY_MUSIC: 'PlayMusic',
      PlayStatus: 'PlayStatus'
    }),
    timeupdate (e) { //更新播放时间
        this.currentTime = e.target.currentTime
      },
    format (interval) { //格式化时间
        interval = Math.floor(interval)
        const minute = Math.floor(interval / 60)
        const second = Math.floor(interval % 60) < 10 ? `0${Math.floor(interval % 60)}` : Math.floor(interval % 60)
        return `${minute}:${second}`
      },
    Mode () {//监听了播放模式的循环
        this.$store.dispatch('PlayModeNum')
      },
    Close () { //关闭播放页面
        this.$refs.scrollDiv2.style.transform = 'translate3d(0,100%,0)'
        this.$refs.scrollDiv.style.opacity = '0'
        this.$refs.scrollDiv.style.zIndex = '-1'
      },
    Show (e) {//展开播放页面
        this.$refs.scrollDiv2.style.transform = 'translate3d(0,0,0)'
        this.$refs.scrollDiv.style.opacity = '1'
        this.$refs.scrollDiv.style.zIndex = '100'
      },
    Pause () {//暂停
        let audio = this.$refs.Audio
        if (!audio.paused) {
          audio.pause()
          this.PlayStatus({
            status: false
          })
        } else {
          audio.play()
          this.PlayStatus({
            status: true
          })
        }
        console.log(this.playing)
      },
    PlayMusic (index, id) { //获取点击选中的播放
        this.PLAY_MUSIC({
          index: index,
          id: id
        })
      },
    PlayInfoHide () { //播放详情页面隐藏
        this.PlayMusicInfoShow = false
      },
    PlayInfoShow () { //播发详情页面显示
      this.PlayMusicInfoShow = true
    },
    TouchStart (e) {
      this.touches.Start = true //开始滑动
      const touch = e.touches[0]
      this.touches.startX = touch.pageX //滑动的左距离
      this.touches.startY = touch.pageX //滑动的上下距离
    },
    TouchMove (e) {
      if (!this.touches.Start) return  //如果在开始的时候没有设置成true  那么就返回出去 不进行滑动
      //进行获取滑动的距离
      const touch = e.touches[0]
      let MoveX = touch.pageX - this.touches.startX //滑动偏移的距离
      let MoveY = touch.pageY - this.touches.startY
      //滑动的时候 有可能从上向下滑动， 那么就需要返回 不就行任何操作 Math.abs() 返回绝对值， 如果说值为负数也会返回正值
      if (Math.abs(MoveY) > Math.abs(MoveX)) return
    },
    TouchEnd (e) {}
  },
  watch: {
    processWidth (newWidth) {//监听设置播放的宽度
      if (newWidth > 0) {
        const processbarWidth = this.$refs.processDiv.clientWidth - progressBtnWidth
        const offsetWidth = newWidth * processbarWidth
        this.$refs.progress.style.width = `${offsetWidth}px`
        this.$refs.progressBtn.style['transform'] = `translate3d(${offsetWidth}px,0,0)`
      }
    }
  }
}
</script>
<style scoped>
  .scroll-div {
    position: fixed;
    top: 0;
    left: 0;
    background: rgba(0, 0, 0, 0.3);
    width: 100%;
    height: 100%;
    transition: all .3s;
    z-index: -1;
  }

  .scroll-div2 {
    width: 100%;
    height: auto;
    position: absolute;
    bottom: 0;
    left: 0;
    color: #fff;
    transform: translate3d(0, 100%, 0);
    transition: all .3s;
  }

  .close, .scroll-music-title {
    width: 100%;
    height: 60px;
    background: #cccccc;
    line-height: 60px;
    background: rgba(21, 39, 31, .9);
    font-size: 14px;
  }

  .scroll-music-title {
    height: 50px;
    line-height: 50px
  }

  .scroll-div2 li {
    line-height: 40px;
    height: 40px;
    width: 100%;
    padding: 0 15px;
    border-bottom: 1px solid #999;
  }

  .scroll-div2 li small {
    font-size: 12px
  }

  .scroll-music-title .icon {
    font-size: 20px;
    vertical-align: middle;
  }

  .scroll-music-title {
    display: flex;
    padding: 0 15px
  }

  .scroll-music-title > div {
    flex: 1;
  }

  .scroll-music-title > div:first-child {
    flex: 5;
    text-align: left
  }

  .scroll-music-title > div:first-child span {
    display: inline-block;
    line-height: 50px;
    height: 50px;
  }

  .scroll-list {
    height: 400px;
    background: rgba(21, 39, 31, 1);
    text-align: left;
  }

  .scroll-list > div {
    height: 100%;
    overflow: hidden
  }

  .music .iconfont {
    font-size: 20px;
    width: 30px;
    height: 30px;
    float: right;
    text-align: center;
  }

  .active span {
    color: #31c27c
  }

  .smt_left > div {
    width: 120px
  }

  #play-music-info {
    position: fixed;
    width: 100%;
    height: 100%;
    top: 0;
    left: 0;
    color: #fff;
    background: #000000;
  }

  .play-music-bg {
    position: absolute;
    width: 100%;
    height: 100%;
    top: 0;
    left: 0;
    background-size: 130% 130%;
    -ms-filter: blur(10px);
    filter: blur(10px);
  }

  #play-music-info .content {
    position: relative;
    z-index: 50;
    width: 100%;
    height: 100%;
    background-color: rgba(0, 0, 0, 0.4);
  }

  .play-music-info-top {
    position: relative;
    text-align: left;
    line-height: 40px;
    height: 40px;
    transition: all 0.4s cubic-bezier(0.86, 0.18, 0.82, 1.32)
  }

  .play-music-info-top .icon {
    font-size: 20px;
    width: 40px;
    height: 40px;
    display: inline-block;
    line-height: 40px;
    text-align: center;
  }

  .icon-xiala2 {
    float: left;
  }

  .icon-listmore {
    position: absolute;
    right: 0;
    top: 0;
  }

  .play-music-info-top div {
    display: block;
    text-align: center;
    margin: 0 40px;
  }

  .play-music-info-center {
    position: fixed;
    left: 0;
    top: 40px;
    bottom: 150px;
    width: 100%;
    height: 100%;
  }

  .play-music-info-image {
    width: 100%;
    height: 100%;
  }

  .play-music-info-image p {
    line-height: 18px;
    font-size: 12px;
    margin-bottom: 1%;
  }

  .play-music-info-image p:nth-child(2) {
    margin-bottom: 5%;
    line-height: 20px;
  }

  .play-music-info-image p:nth-child(2) span {
    border: 1px solid #fff;
    padding: 2px 8px;
    border-radius: 5px;
  }

  .play-music-info-image img {
    width: 100%;
    height: 100%;
    border-radius: 50%;
    border: 10px solid rgba(255, 255, 255, 0.1);
    border-radius: 50%
  }

  .song-image {
    margin: 3% 10%;
    margin-bottom: 20px;
    animation: rotateImg 45s infinite linear;
  }

  .button-show {
    width: 100%;
    display: inline-block;
  }

  .button-show li {
    width: 10px;
    height: 10px;
    border-radius: 50%;
    background: rgba(255, 255, 255, .3);
    display: inline-block;
  }

  .button-show li.active {
    background: #fff
  }

  .play-music-info-bottom {
    position: absolute;
    bottom: 0;
    left: 0;
    height: 100px;
    width: 100%;
    transition: all 0.4s cubic-bezier(0.86, 0.18, 0.82, 1.32)
  }

  .play-music-button-cut {
    display: inline-block;
  }

  .play-music-info-handler {
    text-align: left;
    padding: 0 20px;
    display: flex;
  }

  .play-music-info-handler .icon {
    font-size: 24px;
    flex: 1;
    vertical-align: middle;
    line-height: 30px;
  }

  .play-music-info-handler .icon-shangyishou,
  .play-music-info-handler .icon-play-circle,
  .play-music-info-handler .icon-zanting,
  .play-music-info-handler .icon-xiayishou {
    color: #31c27c;
    font-size: 30px
  }

  .play-music-info-handler .icon-play-circle,
  .play-music-info-handler .icon-zanting {
    font-size: 40px;
    text-align: center;
    padding: 0 20px;
  }

  .play-music-info-time {
    display: flex;
    padding: 0 20px;
    margin-bottom: 20px;
    line-height: 30px;
  }

  .time-start, .time-end {
    flex: 0 0 30px
  }

  .play-music-info-time > div {
    flex: 1;
    height: 3px;
    background: rgba(0, 0, 0, 0.5);
    margin: 0 10px;
    top: 13px;
    position: relative;
    border-radius: 5px;
  }
  .process-bar2{
    width: 0;
    height: 3px;
    background: #31c27c;
  }
  .process-bar-btn{
    width: 16px;
    height: 16px;
    background: #31c27c;
    border-radius: 50%;
    position: relative;
    top: -9px;
    left: 0;
  }
  .play-music-info-image div.playing {
    animation-play-state: paused;
    -webkit-animation-play-state: paused;
  }
  .fade-enter-active,
  .fade-leave-active,
  .content-enter-active,
  .content-leave-active,
  .image-active-active,
  .image-leave-active,
  .lyric-active-active,
  .lyric-leave-active{
    transition: all .5s;
  }
  .fade-enter-active .play-music-info-top,
  .fade-leave-active .play-music-info-top,
  .fade-enter-active .play-music-info-bottom,
  .fade-leave-active .play-music-info-bottom{transition: all .5s cubic-bezier(0.86, 0.18, 0.82, 1.32);}
  .fade-enter, .fade-leave-to /* .fade-leave-active below version 2.1.8 */ {
    opacity: 0;
  }
  .fade-enter .play-music-info-top,
  .fade-leave-to .play-music-info-top{
    transform: translate3d(0, -100px, 0)
  }
  .fade-enter .play-music-info-bottom,
  .fade-leave-to .play-music-info-bottom{
    transform: translate3d(0, 100px, 0)
  }

  @keyframes rotateImg {
    from {
      transform: rotate(0deg)
    }
    to {
      transform: rotate(360deg)
    }
  }

  @media screen and (-webkit-min-device-pixel-ratio: 2) {
    .scroll-div2 li {
      border-bottom: 0.5px solid #999
    }
  }

  @media screen and (-webkit-min-device-pixel-ratio: 3) {
    .scroll-div2 li {
      border-bottom: 0.333333px solid #999
    }
  }
</style>
