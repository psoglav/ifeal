<template>
  <div class="chat-participant">
    <div v-show="!me" ref="pName" class="chat-participant-name">{{ name }}</div>
    <div ref="pAvatar" :class="{ online }" class="chat-participant-avatar">
      <div ref="hitLight" class="chat-participant-avatar_hit-light"></div>
    </div>
    <div v-show="!me" ref="pStatus" class="chat-participant-status">
      {{ computedStatus }}
      <div
        v-show="waiting"
        ref="pStatusWaitingSuffix"
        class="chat-participant-status_waiting-suffix"
      >
        <span>.</span>
        <span>.</span>
        <span>.</span>
      </div>
      <div
        v-show="!waiting"
        ref="pStatusIdleSuffix"
        class="chat-participant-status_idle-suffix"
      >
        <span>.</span>
      </div>
    </div>
  </div>
</template>

<script>
import anime from 'animejs'

export default {
  name: 'ChatParticipant',
  props: {
    name: {
      type: String,
      default: 'Stranger',
    },
    me: {
      type: Boolean,
      default: false,
    },
    lightColor: {
      type: String,
      default: '#bbbbff',
    },
    // status: {
    // type: String,
    // default: 'out',
    // },
  },
  data: () => ({
    statuses: {
      typing: 'is typing',
      out: 'is out',
      staring: 'is staring',
    },
    status: 'out',
    handledStatus: '',
    waitingStatuses: ['typing'],
    waitingAnimation: null,
    idleAnimation: null,
    staringAvatarAnimation: null,
    onlineAnimation: null,
    offlineAnimation: null,
    mouseControlledLook: false,
  }),
  watch: {
    status: {
      immediate: true,
      handler(val) {
        const self = this

        anime({
          targets: self.$refs.pStatus,
          opacity: 0,
          scale: 0.5,
          translateY: [0, -15],
          duration: 100,
          complete() {
            self.handledStatus = val
            self.$refs.pStatus.className = 'chat-participant-status ' + val

            anime({
              targets: self.$refs.pStatus,
              opacity: 1,
              scale: [1.1, 1],
              translateY: [15, 0],
              duration: 400,
            })

            self.waitingAnimation.restart()

            if (!self.online) {
              self.idleAnimation.restart()
              self.idleAnimation.pause()
            }

            self.handleStatus(val)
          },
        })
      },
    },
  },
  computed: {
    online() {
      return this.status != 'out'
    },
    computedStatus() {
      return this.statuses[this.handledStatus]
    },
    waiting() {
      return this.waitingStatuses.includes(this.status)
    },
  },
  methods: {
    distance(x1, x2, y1, y2) {
      var a = x1 - x2
      var b = y1 - y2

      return Math.sqrt(a * a + b * b)
    },
    handleStatus(status) {
      if (status != 'staring') {
        this.idleAnimation.pause()
        this.stopAnimatingStaringAvatar()
      } else {
        this.startAnimatingStaringAvatar()
      }

      this.idleAnimation.restart()
      this.adjustLightToOnlineStatus()
    },
    defineAnimations() {
      let self = this

      this.waitingAnimation = anime({
        targets: self.$refs.pStatusWaitingSuffix.children,
        opacity: [0, 1, 0],
        duration: 2000,
        delay: anime.stagger(150),
        loop: true,
      })

      this.idleAnimation = anime({
        targets: self.$refs.pStatusIdleSuffix.children,
        opacity: [1, 0, 1, 0, 1, 1, 1, 1],
        duration: 1000,
        // loop: true,
      })
    },
    startAnimatingStaringAvatar() {
      if (this.mouseControlledLook) return

      const self = this

      if (!this.staringAvatarAnimation) {
        this.staringAvatarAnimation = anime({
          targets: self.$refs.pAvatar,
          boxShadow() {
            const [x, y] = [anime.random(-25, 25), anime.random(-5, 5)]
            const [blur, spread] = [anime.random(50, 85), anime.random(5, 8)]

            return `0 0 15px 3px ${self.lightColor}, ${x}px ${y}px ${blur}px ${spread}px ${self.lightColor}`
          },
          duration: anime.random(800, 1000),
          complete() {
            self.staringAvatarAnimation = null
            if (!self.mouseControlledLook) self.startAnimatingStaringAvatar()
          },
        })
      } else {
        this.staringAvatarAnimation.play()
      }
    },
    stopAnimatingStaringAvatar() {
      if (this.staringAvatarAnimation) {
        this.staringAvatarAnimation.pause()
      }
    },
    destroyStaringAvatarAnimation() {
      this.stopAnimatingStaringAvatar()
      this.staringAvatarAnimation = null
    },
    adjustLightToOnlineStatus() {
      if (this.online) {
        this.$refs.pAvatar.style.transition = 'box-shadow .2s'
        this.$refs.pAvatar.style.boxShadow = `0 0 7px 5px ${this.lightColor}`
      } else {
        this.$refs.pAvatar.style.transition = 'box-shadow .4s'
        this.$refs.pAvatar.style.boxShadow = `0 0 200px 10px ${this.lightColor}00`
      }
    },
    mouseMoveHandler(e) {
      this.mouseControlledLook = true
      this.destroyStaringAvatarAnimation()

      if (this.status != 'staring') {
        this.adjustLightToOnlineStatus()
        return
      }

      this.$refs.pAvatar.style.transition = 'box-shadow 0.05s'

      const avatarRect = this.$refs.pAvatar.getBoundingClientRect()
      const aw = avatarRect.width
      const ah = avatarRect.height
      const ax = avatarRect.x + aw / 2
      const ay = avatarRect.y + ah / 2
      const mx = e.clientX - ax
      const my = e.clientY - ay

      const dist = this.distance(e.clientX, ax, e.clientY, ay)

      this.$refs.pAvatar.style.boxShadow = `
      ${mx / 30}px ${my / 30}px ${dist / 12 + 5}px 5px ${this.lightColor},
        0 0 25px 3px ${this.lightColor}`
    },
    hit() {
      const self = this

      setTimeout(() => {
        this.$refs.hitLight.style.boxShadow = `0 0 20px 1px ${this.lightColor}`
        setTimeout(() => {
          self.$refs.hitLight.style.transition = 'box-shadow .8s'
          self.$refs.hitLight.style.boxShadow = `0 0 360px 100px ${this.lightColor}00`
        }, 100)
      }, 20)
    },
    hitParticipant() {
      if (!this.me) {
        this.hit()
      }
    },
    hitMe() {
      if (this.me) {
        this.hit()
      }
    },
  },
  created() {
    this.$root.$on('hit-participant', this.hitParticipant)
    this.$root.$on('hit-me', this.hitMe)
  },
  mounted() {
    let self = this

    if (this.me) {
      this.status = 'staring'
      document.addEventListener('mousemove', this.mouseMoveHandler)
    } else {
      setTimeout(function() {
        self.status = 'staring'
        setTimeout(function() {
          self.status = 'typing'
          setTimeout(function() {
            self.$root.$emit('hit-me')
            self.status = 'staring'
            setTimeout(function() {
              self.status = 'typing'
              setTimeout(function() {
                self.$root.$emit('hit-me')
                self.status = 'out'
              }, 5000)
            }, 1000)
          }, 5000)
        }, 8000)
      }, 3000)
    }

    this.defineAnimations()
  },
  beforeDestroy() {
    if (this.me) {
      document.removeEventListener('mousemove', this.mouseMoveHandler)
    }

    this.$root.$off('hit-participant', this.hitParticipant)
    this.$root.$off('hit-me', this.hitMe)
  },
}
</script>

<style lang="scss">
@import '@/scss/colors';

.chat-participant {
  display: grid;
  position: relative;
  width: 300px;
  margin: auto;
  grid-template-areas: 'name avatar status';
  grid-template-columns: 1fr 90px 1fr;
  color: white;
  align-items: center;
  justify-items: center;
  font-family: Montserrat;
  z-index: 10;

  &-name {
    grid-area: name;
    width: 100%;
    font-weight: 500;
    text-align: right;
  }

  &-avatar {
    grid-area: avatar;
    width: 60px;
    height: 60px;
    border-radius: 100%;
    background-color: #fff;
    position: relative;
    // transition: box-shadow 0.4s;

    &_hit-light {
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      border-radius: 100%;
      box-shadow: 0 0 100px 0 $light;
    }

    &.online {
      box-shadow: 0 0 0px 0px #bbbbff;
    }
  }

  &-status {
    grid-area: status;
    opacity: 1;
    width: 100%;
    position: relative;
    text-align: left;
    z-index: 100;

    &_idle-suffix,
    &_waiting-suffix {
      margin-left: -3px;
      display: inline;
    }

    &.out {
      color: #ccccff99;
    }
  }
}
</style>
