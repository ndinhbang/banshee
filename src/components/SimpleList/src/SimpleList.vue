<script>
export default {
  name: 'BansheeSimpleList',
  functional: true,
  props: {
    items: {
      type: Array,
      required: true
    }
  },
  render (h, { props, data }) {
    return props.items.map((item, index) => {
      return data.scopedSlots.default({
        item,
        index
      })
    })
  }
}
</script>
