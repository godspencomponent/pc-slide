<template>
  <div class="component">
    <el-carousel :height="height">
      <el-carousel-item v-for="item in 4" :key="item">
        <h3>{{ item }}</h3>
      </el-carousel-item>
    </el-carousel>
  </div>
</template>

<script>
  import {VueExtend} from 'godspen-lib'

  export default {
    mixins: [VueExtend.mixin],
    name: 'pc-slide',
    label: process.env.LABEL,
    style: process.env.STYLE,
    data () {
      return {
        height: '',
      }
    },
    props: {
    },
    slots: {
    },
    editorMethods: {
    },
    mounted () {
      this.watchHeight()
    },
    methods: {
      watchHeight () {
        let $style = process.env.NODE_ENV !== 'production' ? '$parent.$parent.style.height' : '$parent.nodeInfo.style.height'
        this.$watch($style, () => this.$nextTick(() => {
          this.height = this.$el.clientHeight + 'px'
        }), { immediate: true })
      }
    }
  }
</script>

<style lang="stylus" rel="stylesheet/stylus" type="text/stylus" scoped>
  .component {
    width: 100%;
    height: 100%;
    background-color #f88
  }
</style>
