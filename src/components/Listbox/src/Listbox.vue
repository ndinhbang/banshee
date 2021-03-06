<script>
import BansheeListboxItem from './ListboxItem'
import toPascal from '@/utils/hyphenToPascal'

export default {
  name: 'BansheeListbox',
  props: {
    defaultFocus: {
      type: [String, Number],
      default: 0
    },
    defaultSelected: {
      type: [Array, String, Number],
      default: null
    },
    items: {
      type: Array,
      required: true,
      default: null
    },
    itemKey: {
      type: String,
      default: 'id'
    },
    tag: {
      type: String,
      default: 'div'
    }
  },
  data: () => ({
    focusedIndex: 0,
    selected: []
  }),
  computed: {
    currentItem () {
      return this.items[this.focusedIndex]
    },
    currentItemDOM () {
      return this.$el.querySelectorAll('[role="option"]')[this.focusedIndex]
    }
  },
  mounted () {
    if (this.defaultFocus) {
      this.focusedIndex = parseInt(this.defaultFocus)
    }

    this._handleDefaultSelected()
  },
  methods: {
    _handleDefaultSelected () {
      if (!Array.isArray(this.defaultSelected)) {
        const index = parseInt(this.defaultSelected)
        const item = this.items[index]

        if (item) this.selected.push(item)
      } else {
        this.selected = this.defaultSelected.map(i => this.items[i])
      }
    },
    _removeFromSelected () {
      const index = this.selected.indexOf(this.currentItem)
      this.selected = this.selected.filter((x, i) => i !== index)

      this.$emit('onRemoveItem', {
        removed: this.currentItem,
        selected: this.selected
      })
    },
    handleEvent ({ keyCode }) {
      switch (keyCode) {
        case 13: /** Enter key */
          return this.selectItem()
        case 35: /** End key */
          return this.scrollBottom()
        case 36: /** Home key */
          return this.scrollTop()
        case 38: /** Up arrow */
          return this.up()
        case 40: /** Down arrow */
          return this.down()
      }
    },
    down () {
      this.focusItem(this.focusedIndex + 1)

      if (this.focusedIndex > this.items.length - 1) {
        this.focusItem(0)
      }

      this.scrollIntoView()
    },
    focusItem (index) {
      this.focusedIndex = index
      this.$emit('onFocus', {
        index,
        item: this.items[index]
      })
    },
    selectItem () {
      if (this.currentItem && !this.selected.includes(this.currentItem)) {
        this.selected.push(this.currentItem)

        this.$emit('onSelectItem', {
          item: this.currentItem,
          selected: this.selected
        })
      } else {
        this._removeFromSelected()
      }
    },
    scrollBottom () {
      this.focusItem(this.items.length - 1)
      this.scrollIntoView()
    },
    scrollIntoView () {
      this.$nextTick(() => {
        this.currentItemDOM.scrollIntoView()
      })
    },
    scrollTop () {
      this.focusItem(0)
      this.scrollIntoView()
    },
    transfer () {
      this.$emit('onTransfer', {
        selected: this.selected
      })

      this.$el.focus()
      this.selected = []
    },
    up () {
      this.focusItem(this.focusedIndex - 1)

      if (this.focusedIndex < 0) {
        this.focusItem(this.items.length - 1)
      }

      this.scrollIntoView()
    }
  },
  render (h) {
    const children = this.$slots.default.map(child => {
      const options = child.componentOptions

      if (options && toPascal(options.tag) === BansheeListboxItem.name) {
        return this.items.map((item, index) => {
          let key

          // If item is not an object
          if (item[this.itemKey]) {
            key = item[this.itemKey]
          } else {
            key = `${item}-${index}`
          }

          return h(BansheeListboxItem, {
            props: {
              item,
              index,
              focus: this.focusItem,
              focused: this.focusedIndex,
              selected: this.selected,
              selectItem: this.selectItem,
              ...options.propsData
            },
            key,
            ...child.data
          })
        })
      }

      return child
    })

    const scopedSlot = this.$scopedSlots.default
      ? this.$scopedSlots.default({
        items: this.items,
        focus: this.focusItem,
        focused: this.focusedIndex,
        selectItem: this.selectItem,
        selectedItems: this.selected,
        total: this.items.length,
        totalSelected: this.selected.length,
        transfer: this.transfer
      })
      : null

    return h(this.tag, {
      attrs: {
        role: 'listbox',
        tabindex: 0
      },
      on: {
        dblclick: this.selectItem,
        keydown: this.handleEvent
      }
    }, [children, scopedSlot])
  }
}
</script>
