<template>
<div class="veui-accordion">
  <slot/>
</div>
</template>

<script>
import ui from '../mixins/ui'
import { makeCoupledParent } from '../mixins/coupled'
import { clone } from 'lodash'

let accordion = makeCoupledParent({
  type: 'accordion',
  childrenKey: 'items'
})

export default {
  name: 'veui-accordion',
  uiTypes: ['accordion'],
  mixins: [ui, accordion],
  props: {
    multiple: Boolean,
    disabled: Boolean,
    expanded: [Number, String, Array]
  },
  data () {
    let expanded = this.multiple ? clone(this.expanded || []) : null
    return {
      items: [],
      localExpanded: expanded
    }
  },
  computed: {
    realExpanded () {
      return this.expanded === undefined ? this.localExpanded : this.expanded
    }
  },
  watch: {
    expanded (val) {
      this.localExpanded = clone(val) || null
    }
  },
  methods: {
    toggleById (id) {
      let item = this.findById(id)
      let key = item.name || item.id
      let expanded = clone(this.realExpanded)
      let expand = false
      if (!this.multiple) {
        if (expanded === key) {
          expanded = null
        } else {
          expanded = key
          expand = true
        }
      } else {
        let index = expanded.indexOf(key)
        if (index !== -1) {
          expanded.splice(index, 1)
        } else {
          expanded.push(key)
          expand = true
        }
      }
      this.localExpanded = expanded
      this.$emit('update:expanded', expanded)
      this.$emit('toggle', expand, key, expanded)
    }
  }
}
</script>
