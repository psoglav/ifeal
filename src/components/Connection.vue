<template>
  <div class="connections">
    <div ref="oLine" class="connections-opposite-line">
      <span v-for="i in $range(connectionSegments)" :key="i"></span>
    </div>
    <div ref="mLine" class="connections-my-line">
      <span v-for="i in $range(connectionSegments)" :key="i"></span>
    </div>
  </div>
</template>

<script>
import anime from 'animejs'

export default {
  name: 'connection',
  props: {
    lightColor: {
      type: String,
      default: '#bbbbff',
    },
  },
  data() {
    return {
      connectionSegments: 10,
    }
  },
  methods: {
    hitParticipant() {
      const self = this
      const c = this.lightColor

      anime({
        targets: self.$refs.mLine.children,
        boxShadow: [
          `0 0 100px 0px ${c}`,
          `0 0 88px 6px ${c}`,
          `0 0 100px 0px ${c}`,
        ],
        duration: 150,
        direction: 'reverse',
        autoplay: false,
        delay: anime.stagger(10),
      })
    },
    hitMe() {
      const self = this
      const c = this.lightColor

      anime({
        targets: self.$refs.mLine.children,
        boxShadow: [
          `0 0 100px 0px ${c}`,
          `0 0 88px 6px ${c}`,
          `0 0 100px 0px ${c}`,
        ],
        duration: 150,
        autoplay: true,
        delay: anime.stagger(10),
      })
    },
  },
  created() {
    this.$root.$on('hit-participant', this.hitParticipant)
    this.$root.$on('hit-me', this.hitMe)
  },
  beforeDestroy() {
    this.$root.$off('hit-participant', this.hitParticipant)
    this.$root.$off('hit-me', this.hitMe)
  },
  mounted() {
    this.$root.$emit('hit-me')
  },
}
</script>

<style lang="scss" scoped>
$segments: 10;

.connections {
  position: absolute;
  z-index: 0;
  left: 0;
  width: 100%;
  height: 100%;
  padding: 85px 0;

  &-opposite-line,
  &-my-line {
    position: absolute;
    height: calc(100% - 140px);
    // border-left: 1px solid #bbbbff99;

    display: grid;
    grid-template-rows: repeat(auto-fill, 1fr);

    span {
      width: 0;
      box-shadow: 0 0 100px 0px #bbbbff;
      margin-left: -1px;
      height: 100%;
    }

    // @for $i from 1 through $segments {
    // span:nth-child(#{$i}) {
    // opacity: (1 / $i);
    // }
    // }
  }

  &-opposite-line {
    top: 70px;
    left: calc(50% - 12px);
    height: calc(100% - 180px);
    // animation-name: transmission;
    animation-duration: 1.5s;
    animation-timing-function: ease-in-out;
    animation-iteration-count: infinite;
  }

  &-my-line {
    bottom: 70px;
    left: calc(50% + 12px);
    height: calc(100% - 180px);
  }
}
</style>
