<style lang="scss">
  .ar-player {
    width: 228px;
    height: unset;
    border: 0;
    border-radius: 0;
    display: flex;
    flex-direction: row;
    align-items: center;
    justify-content: left;
    background-color: unset;
    font-family: 'Roboto', sans-serif;

    svg,
    img {
      display: block;
    }

    .ar-volume {
      display: none;
    }

    .ar-player-bar {
      border: 1px solid #E8E8E8;
      border-radius: 24px;
      margin: 0 0 0 5px;

      .ar-line-control__head {
        background-color: #36b37e;
      }

      & > .ar-player__progress {
        width: 100px;
      }
    }

    &-bar {
      display: flex;
      align-items: center;
      height: 38px;
      margin: 0 5px;
      border: none !important;
      padding: 0 0 0 5px;
    }

    &-actions {
      width: auto;
      display: flex;
      align-items: center;
      justify-content: space-around;
    }

    .ar-icon {
      border: none;
    }

    .ar-icon__lg {
      width: 30px;
      height: 30px;
      line-height: 30px;
      box-shadow: none;
      fill: #ffffff;
      background-color: #36b37e !important;
    }

    &__progress {
      width: 160px;
      margin: 0 8px;
    }

    &__time {
      font-size: 14px;
      width: auto;
      color: #8492a6;
    }

    &__play {
      width: 45px;
      height: 45px;
      background-color: #FFFFFF;
      box-shadow: 0 2px 11px 11px rgba(0,0,0,0.07);

      &--active {
        fill: white !important;
        background-color: #36b37e !important;

        &:hover {
          fill: white !important;
          background-color: #36b37e !important;
        }
      }
    }
  }

  @import '../scss/icons';
</style>

<template>
  <div class="ar-player-button">
    <div class="ar-player-actions">
      <icon-button
        id="play"
        class="ar-icon ar-icon__lg ar-player__play"
        :name="playBtnIcon"
        :class="{'ar-player__play--active': isPlaying}"
        @click.native="playback"/>
    </div>

    <div class="ar-player-bar">
      <div class="ar-player__time">{{playedTime}}</div>
      <line-control
        class="ar-player__progress"
        ref-id="progress"
        :percentage="progress"
        @change-linehead="_onUpdateProgress"/>
      <div class="ar-player__time">{{duration}}</div>
      <volume-control @change-volume="_onChangeVolume"/>
    </div>

    <audio :id="playerUniqId" :src="audioSource"></audio>
  </div>
</template>

<script>
  import IconButton    from './icon-button'
  import LineControl   from './line-control'
  import VolumeControl from './volume-control'
  import { convertTimeMMSS } from '@/lib/utils'

  export default {
    props: {
      src      : { type: String },
      record   : { type: Object },
      filename : { type: String }
    },
    data () {
      return {
        isPlaying  : false,
        duration   : convertTimeMMSS(0),
        playedTime : convertTimeMMSS(0),
        progress   : 0
      }
    },
    components: {
      IconButton,
      LineControl,
      VolumeControl
    },
    mounted: function() {
      this.player = document.getElementById(this.playerUniqId)

      this.player.addEventListener('ended', () => {
        this.isPlaying = false
      })

      this.player.addEventListener('loadeddata', (ev) => {
        this._resetProgress()
        this.duration = convertTimeMMSS(this.player.duration)
      })

      this.player.addEventListener('timeupdate', this._onTimeUpdate)

      this.$eventBus.$on('remove-record', () => {
        this._resetProgress()
      })
    },
    computed: {
      audioSource () {
        const url = this.src || this.record.url
        if (url) {
          return url
        } else {
          this._resetProgress()
        }
      },
      playBtnIcon () {
        return this.isPlaying ? 'pause' : 'play'
      },
      playerUniqId () {
        return `audio-player${this._uid}`
      }
    },
    methods: {
      playback () {
        if (!this.audioSource) {
          return
        }

        if (this.isPlaying) {
          this.player.pause()
        } else {
          setTimeout(() => { this.player.play() }, 0)
        }

        this.isPlaying = !this.isPlaying
      },
      _resetProgress () {
        if (this.isPlaying) {
          this.player.pause()
        }

        this.duration   = convertTimeMMSS(0)
        this.playedTime = convertTimeMMSS(0)
        this.progress   = 0
        this.isPlaying  = false
      },
      _onTimeUpdate () {
        this.playedTime = convertTimeMMSS(this.player.currentTime)
        this.progress = (this.player.currentTime / this.player.duration) * 100
      },
      _onUpdateProgress (pos) {
        if (pos) {
          this.player.currentTime = pos * this.player.duration
        }
      },
      _onChangeVolume (val) {
        if (val) {
          this.player.volume = val
        }
      }
    }
  }
</script>