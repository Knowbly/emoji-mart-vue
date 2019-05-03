<script>

import { getData, getSanitizedData, unifiedToNative, getBgPosition } from '../../utils'
import { uncompress } from '../../utils/data'
import { EmojiProps } from '../../utils/shared-props'

const SHEET_COLUMNS = 52

export default {
  props: {
    ...EmojiProps,
    data: {
      type: Object,
      required: true
    }
  },
  functional: true,
  render(h, ctx) {
    let { data, props, listeners } = ctx

    let emojiData = {}
    const mutableData = props.data.compressed ? uncompress(props.data) : props.data
    if (props.emoji) {
      emojiData = getData(props.emoji, props.skin, props.set, mutableData)
    }
    const hasEmoji = emojiData['has_img_' + props.set]
    const isNative = props.native
    const isCustom = emojiData.custom

    if (isCustom || isNative || hasEmoji || props.fallback) {
      const customEmojiStyles = {
        display: 'inline-block',
        width: props.size + 'px',
        height: props.size + 'px',
        backgroundImage: 'url(' + emojiData.imageUrl + ')',
        backgroundSize: '100%',
      }
      let child = null;
      if (isCustom) {
        child = h('span', { attrs: {title: props.title}, style: {
          display: 'inline-block',
          width: props.size + 'px',
          height: props.size + 'px',
          backgroundImage: 'url(' + emojiData.imageUrl + ')',
          backgroundSize: '100%',
        } })
      } else if (isNative) {
        let style = { fontSize: props.size + 'px' }
        if (props.forceSize) {
          style.display = 'inline-block'
          style.width = props.size + 'px'
          style.height = props.size + 'px'
        }
        const nativeEmoji = emojiData.unified ? unifiedToNative(emojiData.unified) : ''
        child = h('span' , { attrs: {title: props.title}, style }, nativeEmoji)
      } else if (hasEmoji) {
        let fallbackEmojiStyles = null
        if (isCustom) {
          fallbackEmojiStyles = customEmojiStyles
        } else if (hasEmoji) {
          fallbackEmojiStyles = {
            display: 'inline-block',
            width: props.size + 'px',
            height: props.size + 'px',
            backgroundImage: 'url(' + props.backgroundImageFn(props.set, props.sheetSize) + ')',
            backgroundSize: (100 * SHEET_COLUMNS) + '%',
            backgroundPosition: getBgPosition(emojiData, SHEET_COLUMNS)
          }
        }
        child = h('span', { attrs: {title: props.title}, style: fallbackEmojiStyles })
      } else {
        const fallbackEmoji = props.fallback ? props.fallback(props.emoji) : null
        child = h('span', fallbackEmoji)
      }

      const self = this;
      const sanitizedData = getSanitizedData(props.emoji, props.skin, props.set, mutableData)
      return h('span', { ...data, class: 'emoji-mart-emoji', props, on: {
          click: event => {
            if (!listeners.click) {
              return;
            }
            const emit = listeners.click
            emit(sanitizedData)
          },
          mouseleave: event => {
            if (!listeners.mouseleave) {
              return;
            }
            const emit = listeners.mouseleave
            emit(sanitizedData)
          },
          mouseenter: event => {
            if (!listeners.mouseenter) {
              return;
            }
            const emit = listeners.mouseenter
            emit(sanitizedData)
          },
        }
      }, [child])
    }
    return h('span');
  },
/*
  data() {
    return {
      mutableData: this.data.compressed ? uncompress(this.data) : this.data,
    }
  },
  computed: {
    emojiData() {
      if (!this.emoji) {
        return {}
      }
      return getData(this.emoji, this.skin, this.set, this.mutableData)
    },
    sanitizedData() {
      return getSanitizedData(this.emoji, this.skin, this.set, this.mutableData)
    },
    canRender() {
      return this.isCustom || this.isNative || this.hasEmoji || this.fallback
    },
    isNative() {
      return this.native
    },
    isCustom() {
      return this.emojiData.custom
    },
    hasEmoji() {
      return this.emojiData['has_img_' + this.set]
    },
    nativeEmoji() {
      if (this.emojiData.unified) {
        return unifiedToNative(this.emojiData.unified)
      } else {
        return ''
      }
    },
    fallbackEmoji() {
      return this.fallback ? this.fallback(this.emoji) : null
    },
    nativeEmojiStyles() {
      let styles = { fontSize: this.size + 'px' }

      if (this.forceSize) {
        styles.display = 'inline-block'
        styles.width = this.size + 'px'
        styles.height = this.size + 'px'
      }

      return styles
    },
    fallbackEmojiStyles() {
      if (this.isCustom) {
        return this.customEmojiStyles
      } else if (this.hasEmoji) {
        return {
          display: 'inline-block',
          width: this.size + 'px',
          height: this.size + 'px',
          backgroundImage: 'url(' + props.backgroundImageFn(this.set, this.sheetSize) + ')',
          backgroundSize: (100 * SHEET_COLUMNS) + '%',
          backgroundPosition: getBgPosition(emojiData, SHEET_COLUMNS)
        }
      } else {
        return null
      }
    },
    customEmojiStyles() {
      return {
        display: 'inline-block',
        width: this.size + 'px',
        height: this.size + 'px',
        backgroundImage: 'url(' + this.emojiData.imageUrl + ')',
        backgroundSize: '100%',
      }
    },
    title() {
      return this.tooltip ? this.emojiData.short_names[0] : null
    }
  },
  methods: {
    getPosition() {
      let multiply = 100 / (SHEET_COLUMNS - 1),
          x = multiply * this.emojiData.sheet_x,
          y = multiply * this.emojiData.sheet_y

      return `${x}% ${y}%`
    },
    onClick() {
      this.$emit('click', this.sanitizedData)
    },
    onMouseEnter() {
      this.$emit('mouseenter', this.sanitizedData)
    },
    onMouseLeave() {
      this.$emit('mouseleave', this.sanitizedData)
    }
  }
*/
}

</script>

<style>

.emoji-mart-emoji {
  position: relative;
  display: inline-block;
  font-size: 0;
}

</style>
