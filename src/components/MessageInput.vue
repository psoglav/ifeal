<template>
  <div
    class="message-input common"
    :class="{ focused: inputFocused }"
    @click="focusInput"
  >
    <span :class="{ hidden: input }" class="message-input_placeholder"
      >Start typing...</span
    >
    <div
      ref="input"
      class="message-input_editor common"
      @input="handleInput"
      contenteditable="true"
      spellcheck="false"
      tabindex="0"
    ></div>
    <div
      @click="handleSubmit"
      :class="{ hidden: submitHidden || !inputFocused }"
      class="message-input_submit"
    >
      Send
    </div>
  </div>
</template>

<script>
export default {
  name: 'message-input',
  data() {
    return {
      input: '',
      inputFocused: false,
      initialInputHeight: '',
      prevInputHeight: '',
      inputRect: null,
      submitHidden: false,
    }
  },
  watch: {
    input() {
      const t = this
      setTimeout(() => {
        if (t.prevInputHeight != t.inputHeight()) {
          t.prevInputHeight = t.inputHeight()

          t.submitHidden = t.initialInputHeight != t.inputHeight()
        }
      }, 10)
    },
  },
  methods: {
    focusInput() {
      this.$refs.input.focus()
    },
    handleInput() {
      this.input = this.$refs.input.innerText
      console.log(this.input)
    },
    handleSubmit() {
      if (!this.input) return
      this.$emit('send-message', this.input)
      this.$root.$emit('hit-participant')
      this.$refs.input.innerText = ''
      this.input = ''
    },
    enterNewLine() {
      this.input += '\n'
    },
    inputHeight() {
      this.inputRect = this.$refs.input.getBoundingClientRect()
      return this.inputRect.height
    },
    handleInputFocusEvent() {
      this.inputFocused = true
    },
    handleInputBlurEvent() {
      this.inputFocused = false
    },
    handlePasteEvent(e) {
      const map = {
        '&': '&amp;',
        '<': '&lt;',
        '>': '&gt;',
        '"': '&quot;',
        "'": '&#039;',
      }

      const escape_text = text => text.replace(/[&<>"']/g, m => map[m])

      e.preventDefault()
      var text = (e.originalEvent || e).clipboardData.getData('text/plain')
      document.execCommand('insertHtml', false, escape_text(text))
    },
  },
  beforeDestroy() {
    this.$refs.input.removeEventListener('focus', this.handleInputFocusEvent)
    this.$refs.input.removeEventListener('blur', this.handleInputBlurEvent)
    this.$refs.input.removeEventListener('paste', this.handlePasteEvent)
  },
  mounted() {
    this.$refs.input.addEventListener('focus', this.handleInputFocusEvent)
    this.$refs.input.addEventListener('blur', this.handleInputBlurEvent)
    this.$refs.input.addEventListener('paste', this.handlePasteEvent)

    this.initialInputHeight = this.inputHeight()
    this.prevInputHeight = this.inputHeight()

    this.$refs.input.focus()
  },
}
</script>

<style lang="scss">
@import '@/scss/colors';

.common {
  position: absolute;
  padding: 0.5rem 0;
  padding-left: 1rem;
  padding-right: 0.5rem;
}

.message-input {
  left: 3%;
  padding-left: 1.1rem;
  bottom: 67px;
  z-index: 10;
  width: 40%;
  height: auto;
  z-index: 100;
  max-height: 30vh;

  transition: height 0.1s, box-shadow 0.4s, opacity 0.3s;
  overflow-x: hidden;

  border-radius: 20px;
  backdrop-filter: blur(10px);
  box-shadow: 0 0 10px 2px black, 0 0 10px 1px transparentize($light, 1);
  background-color: #00000033;

  color: transparent;
  font-size: 1rem;
  font-family: 'Open Sans', Montserrat, sans-serif;
  letter-spacing: 0.1rem;
  word-break: break-all;
  text-align: left;
  user-select: none;

  &.focused {
    box-shadow: 0 0 10px 2px black, 0 0 100px 1px transparentize($light, 0.7);
  }

  &_placeholder {
    top: 16px;
    left: 36px;
    position: absolute;
    letter-spacing: 0;
    color: $half-light;
    font-style: italic;
    transition: all 0.4s;

    &.hidden {
      transform: scale(1.3);
      opacity: 0;
    }
  }

  &_editor {
    top: 0;
    left: 0;
    z-index: 10;
    width: 100%;
    min-height: 38px;
    user-select: none;

    position: relative;
    width: 100%;
    height: 100%;
    border: none;
    word-break: break-all;
    color: white;
    background-color: transparent;
    font-family: 'Open Sans', Montserrat, sans-serif;
    font-size: 1rem;

    &:focus {
      outline: none;
    }

    &::selection,
    *::selection {
      background-color: $subtle-light;
    }
  }

  &_submit {
    letter-spacing: 0;
    position: absolute;
    right: 0;
    top: 0;
    height: 100%;
    width: 60px;
    background-color: black;
    cursor: pointer;
    text-align: center;
    color: white;
    display: grid;
    align-content: center;
    font-size: 0.9rem;
    z-index: 300;
    transition: all 0.2s;

    &:hover {
      background-color: #3a3f49;
    }

    &.hidden {
      transition: all 0.8s;
      z-index: 5;
      opacity: 0;
      filter: blur(50px);
    }
  }
}
</style>
