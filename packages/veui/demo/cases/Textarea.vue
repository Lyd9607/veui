<template>
<article>
  <h1><code>&lt;veui-textarea&gt;</code></h1>
  <section>
    <h2>小尺寸 + 显示 3 行</h2>
    <veui-textarea
      v-model="value"
      line-number
      autofocus
      ui="s"
      rows="3"
    />
    <section/>
    <h2>默认尺寸 + 行号 + 自动扩展</h2>
    <veui-textarea
      v-model="value"
      line-number
      select-on-focus
      autoresize
      placeholder="请输入"
    />
    <section/>
    <h2>自动扩展</h2>
    <veui-textarea
      v-model="value"
      autoresize
    />
  </section>
</article>
</template>

<script>
import bus from '../bus'
import { Textarea } from 'veui'

export default {
  name: 'button-demo',
  components: {
    'veui-textarea': Textarea
  },
  data () {
    return {
      value: ''
    }
  },
  mounted () {
    this.$children.forEach(child => {
      child.$on('click', () => {
        bus.$emit('log', child.$el.getAttribute('ui'))
      })
    })
  }
}
</script>

<style scoped></style>
