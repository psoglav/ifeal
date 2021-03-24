<template>
  <div class="chat">
    <chat-participant name="wwwolfie" status="typing" />
    <div class="connection">
      <div class="connection-bottom-line"></div>
      <div class="connection-top-line"></div>
    </div>
    <chat-participant me status="typing" />
  </div>
</template>

<script>
import ChatParticipant from '../components/ChatParticipant.vue'

export default {
  name: 'Chat',
  components: {
    ChatParticipant,
  },
}
</script>

<style lang="scss">
.chat {
  position: relative;
  display: grid;
  height: 100vh;
  padding: 60px 0;
  align-content: space-between;

  .connection {
    position: absolute;
    top: 70px;
    left: 0;
    width: 100%;
    height: 100vh;

    &-bottom-line,
    &-top-line {
      position: absolute;
      height: calc(100% - 140px);
      border-left: 1px solid white;
      border-left-style: dashed;
    }

    &-bottom-line {
      left: calc(50% - 10px);
    }
    &-top-line {
      left: calc(50% + 10px);
    }
  }

  &-participant {
  }
}
</style>
