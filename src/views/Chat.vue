<template>
  <div class="chat">
    <div class="chat_participants">
      <chat-participant name="wwwolfie" status="typing" />
      <connection />
      <chat-participant me status="typing" />
    </div>
    <div class="chat-container">
      <div class="chat-container_messages">
        <div class="chat-container_messages-stack">
          <div class="chat-container_messages-stack_item">
            я был на море
          </div>
          <div class="chat-container_messages-stack_item">
            и море шептало мне своим сладким бризом
          </div>
          <div class="chat-container_messages-stack_item">
            - возвращайся
          </div>
          <div class="chat-container_messages-stack_item">
            - мне одиноко
          </div>
        </div>
        <div class="chat-container_messages-stack">
          <div class="chat-container_messages-stack_item">
            я был на море
          </div>
          <div class="chat-container_messages-stack_item">
            и море шептало мне своим сладким бризом
          </div>
          <div class="chat-container_messages-stack_item">
            - возвращайся
          </div>
          <div class="chat-container_messages-stack_item">
            - мне одиноко
          </div>
        </div>
        <div class="chat-container_messages-stack">
          <div class="chat-container_messages-stack_item">
            я был на море
          </div>
          <div class="chat-container_messages-stack_item">
            и море шептало мне своим сладким бризом
          </div>
          <div class="chat-container_messages-stack_item">
            - возвращайся
          </div>
          <div class="chat-container_messages-stack_item">
            - мне одиноко
          </div>
        </div>
        <div class="chat-container_messages-stack opposite">
          <div class="chat-container_messages-stack_item">
            я был на море
          </div>
          <div class="chat-container_messages-stack_item">
            и море шептало мне своим сладким бризом
          </div>
          <div class="chat-container_messages-stack_item">
            - возвращайся
          </div>
          <div class="chat-container_messages-stack_item">
            - мне одиноко
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import Connection from '~/Connection'
import ChatParticipant from '~/ChatParticipant'

export default {
  name: 'Chat',
  components: {
    Connection,
    ChatParticipant,
  },
  data() {
    return { connectionSegments: 10 }
  },
}
</script>

<style lang="scss">
@import '@/scss/colors';

$msg-min-radius: 0.5rem;
$msg-max-radius: 0.9rem;
$msg-padding: 1rem;
$msg-font-size: 0.95rem;
$msg-bg: $subtle-light;
$msg-fg: $primary-foreground;
$msg-gap: 10px;
$stacks-gap: 30px;

.chat {
  position: relative;
  height: 100vh;
  margin: 0 auto;
  max-width: 1300px;

  &_participants {
    position: relative;
    display: grid;
    min-height: 500px;
    height: 100%;
    align-content: space-between;
    padding: 60px 0;
  }

  &-container {
    position: absolute;
    z-index: 0;
    top: 0;
    height: calc(100% - 80px);
    width: 100%;
    overflow: auto;

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

        &.opposite {
          align-self: flex-end;
        }

        &_item {
          max-width: 400px;
          text-align: left;
          background-color: $msg-bg;
          padding: $msg-padding;
          color: $msg-fg;
          font-size: $msg-font-size;
          font-family: Montserrat;
          align-self: flex-start;
          border-radius: $msg-min-radius 0 0 $msg-min-radius;

          &:last-child {
            border-radius: 0 0 $msg-max-radius $msg-max-radius;
          }

          &:first-child {
            border-radius: $msg-max-radius 0 0 0;
          }
        }

        &.opposite &_item {
          align-self: flex-end;
        }
      }
    }
  }
}
</style>
