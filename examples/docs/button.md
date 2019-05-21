<script>
export default {
  data () {
    return {
      value: '我是按钮'
    }
  },
  methods: {
    click () {
      this.value = '点击过'
    }
  }
}
</script>
## Button
介绍Button的使用
:::demo
``` html
<template>
  <bwt-button
    :value="value"
    @click="click"
  ></bwt-button>
</template>
<script>
export default {
  data () {
    return {
      value: '我是按钮'
    }
  },
  methods: {
    click () {
      this.value = '点击过'
    }
  }
}
</script>
```
:::