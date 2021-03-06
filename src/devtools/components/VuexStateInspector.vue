<template>
  <div>
    <section class="top">
      <div class="buttons">
        <a class="button" @click="copyStateToClipboard">
          <i class="material-icons">content_copy</i>
          <span>Export</span>
          <span class="message"
            transition="slide-up"
            v-show="showStateCopiedMessage">
            (Copied to clipboard!)
          </span>
        </a>
        <a class="button" @click="toggleImportStatePopup">
          <i class="material-icons">content_paste</i>
          <span>Import</span>
        </a>
      </div>

      <div class="import-state" transition="slide-up" v-if="showImportStatePopup">
        <textarea placeholder="Paste state object here to import it..."
          @input="tryImportState"
          @keydown.esc="closeImportStatePopup"></textarea>
        <span class="message invalid-json"
          transition="slide-up"
          v-show="showBadJSONMessage">
          INVALID JSON!
        </span>
      </div>
    </section>
    <div class="vuex-state-inspector">
      <data-field
        v-for="(value, key) of activeState"
        :field="{ key: key, value: value }"
        :depth="0">
      </data-field>
    </div>
  </div>
</template>

<script>
import CircularJSON from 'circular-json-es6'
import DataField from './DataField.vue'
import { activeState } from '../vuex/modules/vuex/getters'
import { importState } from '../vuex/modules/vuex/actions'
import { stringify } from '../../util'
import debounce from 'lodash.debounce'

export default {
  components: {
    DataField
  },
  data () {
    return {
      showStateCopiedMessage: false,
      showBadJSONMessage: false,
      showImportStatePopup: false
    }
  },
  vuex: {
    getters: {
      activeState
    },
    actions: {
      importState
    }
  },
  watch: {
    showImportStatePopup (val) {
      if (val) {
        this.$nextTick(() => {
          this.$el.querySelector('textarea').focus()
        })
      }
    }
  },
  methods: {
    copyStateToClipboard () {
      copyToClipboard(this.activeState.state)
      this.showStateCopiedMessage = true
      window.setTimeout(() => {
        this.showStateCopiedMessage = false
      }, 2000)
    },
    toggleImportStatePopup () {
      if (this.showImportStatePopup) {
        this.closeImportStatePopup()
      } else {
        this.showImportStatePopup = true
      }
    },
    closeImportStatePopup () {
      this.showImportStatePopup = false
    },
    tryImportState: debounce(function (e) {
      const importedStr = e.target.value
      if (importedStr.length === 0) {
        this.showBadJSONMessage = false
      } else {
        try {
          CircularJSON.parse(importedStr) // Try to parse
          this.importState(importedStr)
          this.showBadJSONMessage = false
        } catch (e) {
          this.showBadJSONMessage = true
        }
      }
    }, 250)
  }
}

function copyToClipboard (state) {
  const dummyTextArea = document.createElement('textarea')
  dummyTextArea.textContent = stringify(state)
  document.body.appendChild(dummyTextArea)
  dummyTextArea.select()
  document.execCommand('copy')
  document.body.removeChild(dummyTextArea)
}
</script>

<style lang="stylus" scoped>
$border-color = #e3e3e3
$blue = #44A1FF

.vuex-state-inspector
  padding 15px 20px

.top
  border-bottom 1px solid $border-color
  height 50px
  justify-content: space-between
  font-size 18px
  box-shadow: 0 0 8px rgba(0,0,0,0.15);

.button
  float left
  position relative
  align-items center
  font-size 12px
  color #666
  text-align center
  cursor pointer
  border-right 1px solid $border-color
  transition box-shadow .25s ease, color .2s ease
  height 50px
  line-height 50px
  padding: 0 22px 0 20px
  display inline-block
  vertical-align middle
  i
    margin-right 3px
    vertical-align middle
  span
    white-space nowrap
    vertical-align middle
  &:hover
    box-shadow 0 2px 12px rgba(0,0,0,.1)

.message
  transition all .3s ease
  color $blue

.invalid-json
  right 20px
  left initial
  top 1px
  font-size 12px
  color #c41a16
  background-color #fff

.import-state
  transition all .3s ease
  position absolute
  z-index 1
  left 220px
  right 10px
  top 5px
  box-shadow 4px 4px 6px 0 $border-color
  border 1px solid $border-color
  padding 3px
  background-color #fff

  textarea
    width 100%
    height 100px
    display block
    outline none
    border none
    resize vertical
</style>
