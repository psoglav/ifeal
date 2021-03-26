p<template>
  <div class="container">
    <div ref="messagesContainer" class="container_messages">
      <div
        v-for="(stack, i) in stacks"
        :key="i"
        :class="{ opposite: me.id != stack[0].user }"
        class="container_messages-stack"
      >
        <div class="container_messages-stack_time">
          <span class="expanded">
            {{ formatTime(stack[0].at, true) }}
          </span>
          <span class="collapsed">
            {{ formatTime(stack[0].at) }}
          </span>
        </div>
        <p
          v-for="(message, k) in stack"
          :key="k"
          class="container_messages-stack_item"
          :class="{ stacked: stack.length > 1 }"
        >
          <span class="highlight"></span>
          {{ message.content.text }}
        </p>
      </div>
    </div>
  </div>
</template>

<script>
import moment from 'moment'

export default {
  name: 'MessagesBox',
  data() {
    return {
      stacks: [],
    }
  },
  watch: {
    messages: {
      immediate: true,
      handler(v) {
        let messages = [...v].sort((a, b) => a.at - b.at)

        let ts = 0
        let next = null
        let prev = null

        this.stacks = []

        while (messages.length) {
          let l = this.stacks.length
          next = messages.shift()

          if (!ts || ts + 60000 <= next.at || next.user != prev?.user) {
            ts = next.at
            this.stacks.push([next])
            prev = next
          } else {
            this.stacks[l - 1].push(next)
          }
        }

        // const el = this.$refs.messagesContainer
        // setTimeout(() => {
        // el.scrollTop = 2000
        // }, 100)
      },
    },
  },
  computed: {
    me() {
      return this.$store.getters.me
    },
    user() {
      return this.$store.getters.user
    },
    messages() {
      return this.$store.getters.messages
    },
  },
  methods: {
    formatTime(ts, expand) {
      const d = moment(ts)

      return expand ? d.format('MMMM Do YYYY, h:mm a') : d.format('LT')
    },
  },
  mounted() {
    this.$scrollTo(this.$el)
  }
}
</script>

<style lang="scss">
@import '@/scss/colors';

$msg-bg: linear-gradient(125deg, $darkness, $subtle-light);
$opp-msg-bg: linear-gradient(45deg, $subtle-light, $darkness);
$msg-fg: #eeeeffee;
$msg-min-radius: 5px;
$msg-max-radius: 0.9rem;
$msg-padding: 1rem;
$msg-font-size: 0.95rem;
$msg-gap: 5px;
$stacks-gap: 15px;

$msg-hover-time: 0.4s;

.container {
  position: absolute;
  padding: 80px 0;
  z-index: 0;
  top: 0;
  height: 100%;
  width: 100%;
  overflow: auto;
  overflow-x: hidden;
  background-color: #bbccff17;
  box-shadow: inset 0 0 45px 25px black;

  &_messages {
    width: 100%;
    grid-gap: $stacks-gap;
    padding: 10px;
    display: flex;
    flex-direction: column;
    justify-content: flex-end;

    &-stack {
      width: 43%;
      display: flex;
      flex-direction: column;
      grid-gap: $msg-gap;
      position: relative;
      transform-style: preserve-3d;
      perspective: 1500px;
      perspective-origin: 0% 0%;
      transition: transform 0.2s;

      &:hover {
        transform: translateY(-10px);
      }

      &.opposite {
        align-self: flex-end;
        perspective-origin: 100% 0%;
      }

      &:hover &_item {
        transform: rotateX(-5deg) rotateY(7deg);
      }

      &.opposite:hover &_item {
        transform: rotateX(-5deg) rotateY(-4deg);
      }

      &:hover &_item .highlight {
        transform: translateX(-120%) translateY(-50%) rotate(45deg);
      }

      &.opposite:hover &_item .highlight {
        transform: translateX(120%) translateY(-50%) rotate(45deg);
      }

      &_item {
        max-width: 400px;
        text-align: left;
        background: $msg-bg;
        padding: $msg-padding;
        font-size: $msg-font-size;
        font-family: 'Open Sans', Montserrat, sans-serif;
        font-weight: 500;
        color: $msg-fg;
        align-self: flex-start;
        white-space: pre-wrap;
        hyphens: manual; // doesn't work
        word-break: break-all; // using this instead
        border-radius: 0 $msg-max-radius $msg-max-radius $msg-max-radius;
        overflow: hidden;
        position: relative;
        transition: transform $msg-hover-time;

        .highlight {
          position: absolute;
          top: 10%;
          left: 0;
          border-radius: 100%;
          width: 300px;
          height: 300px;
          opacity: 0.2;
          transform: translateX(150%) translateY(50%) rotate(45deg);
          filter: blur(35px);
          transition: all $msg-hover-time;
          background-color: #fff;
          pointer-events: none;
        }

        &::selection {
          background-color: $subtle-light;
        }

        &.stacked {
          border-radius: 0 $msg-min-radius $msg-min-radius 0;
        }

        &.stacked:last-child {
          border-radius: 0 $msg-min-radius $msg-max-radius $msg-max-radius;
        }

        &.stacked:first-child {
          border-radius: 0 $msg-max-radius $msg-min-radius 0;
        }
      }

      &.opposite &_item {
        align-self: flex-end;
        background: $opp-msg-bg;
        border-radius: $msg-max-radius 0 $msg-max-radius $msg-max-radius;

        .highlight {
          transform: translateX(-120%) translateY(50%) rotate(45deg);
        }

        &.stacked {
          border-radius: $msg-min-radius 0 0 $msg-min-radius;
        }

        &.stacked:last-child {
          border-radius: $msg-min-radius 0 $msg-max-radius $msg-max-radius;
        }

        &.stacked:first-child {
          border-radius: $msg-max-radius 0 0 $msg-min-radius;
        }
      }

      &_time {
        left: 0;
        bottom: -0.2rem;
        position: absolute;
        text-align: left;
        text-transform: uppercase;
        font-size: 0.8rem;
        opacity: 0;
        width: 100%;
        font-family: 'Open Sans', Montserrat, sans-serif;
        transition: opacity 0.1s;
        pointer-events: none;

        .expanded,
        .collapsed {
          display: block;
          position: absolute;
          transition: opacity 0.4s;
          top: 0;
        }
        .expanded {
          opacity: 0;
        }
        .collapsed {
          opacity: 1;
        }
      }

      &.opposite &_time {
        right: 0;
        text-align: right;

        .expanded,
        .collapsed {
          right: 0;
        }
      }

      &:hover &_time {
        opacity: 0.5;
        transition: opacity 0.4s;

        .expanded,
        .collapsed {
          transition-delay: 1s;
        }
        .expanded {
          opacity: 1;
        }
        .collapsed {
          opacity: 0;
        }
      }
    }
  }
}
</style>
