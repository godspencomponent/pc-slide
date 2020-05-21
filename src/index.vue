<template>
  <div class="component carousel">
    <ar-carousel ref="carousel" :autoplay='autoplay' :interval='interval * 1000' 
    :direction='direction' @change='change'
    :indicator-position='indicatorPosition' :loop='loop' :trigger='trigger' :arrow='arrow'
    :type='type' :height="height" :initial-index='initialIndex'>
      <ar-carousel-item v-for="(item,i) in slotslen" :key="item">
        <slot :name="'slot'+i"></slot>
      </ar-carousel-item>
    </ar-carousel>
  </div>
</template>

<script>
  import {VueExtend} from 'godspen-lib'
  import ArCarousel from 'element-ui/lib/carousel'
  import ArCarouselItem from 'element-ui/lib/carousel-item'

  const inClient = process.env.NODE_ENV === 'production' && !/edit[oe]r|preview/.test(window.EDIT_TYPE)

  if (inClient) {
    require('element-ui/lib/theme-chalk/carousel.css')
    require('element-ui/lib/theme-chalk/carousel-item.css')
  }

  export default {
    mixins: [VueExtend.mixin],
    name: 'pc-slide',
    label: process.env.LABEL,
    style: process.env.STYLE,
    components: { ArCarousel, ArCarouselItem },
    data () {
      return {
        height: '',
        slotslen: 0,
        pageMap: {},
        direction: 'horizontal',
        loop: true,
        initialIndex: 0,
      }
    },
    props: {
      showIndicator: {
        type: Boolean,
        editor: {
          label: '显示指示器',
        },
        default: true
      },
      autoplay: {
        type: Boolean,
        editor: {
          label: '自动轮播',
        },
        default: false
      },
      interval: {
        type: Number,
        editor: {
          work () {
            return !!this.autoplay
          },
          label: '自动轮播间隔',
        },
        default: 3
      },
      trigger: {
        type: String,
        editor: {
          label: '切页方式',
          type: 'enum',
          defaultList: ['click', 'hover']
        },
        default: 'click'
      },
      arrow: {
        type: String,
        editor: {
          label: '箭头显示',
          type: 'enum',
          defaultList: ['always', 'hover', 'never']
        },
        default: 'always'
      },
      type: {
        type: String,
        editor: {
          label: '风格',
          type: 'enum',
          defaultList: [ 'card', 'normal' ]
        },
        default: ''
      },
      onchange: {
        type: Array,
        default: function () {
          return []
        },
        editer: {
          label: '切页回调',
          type: 'function'
        }
      }
    },
    slots: {
      style: {
        height: '100%',
        width: '100%',
        display: 'block',
      }
    },
    computed: {
      indicatorPosition () {
        return this.showIndicator ? '' : 'none'
      }
    },
    editorMethods: {
      prev: {
        label: '切换到上一页',
        params: []
      },
      next: {
        label: '切换到下一页',
        params: []
      },
      setActiveItem: {
        label: '切换到指定页',
        params: [
          {
            label: '页数（0 到 n）',
            type: 'number'
          }
        ]
      },
    },
    mounted () {
      this.watchParent()
    },
    methods: {
      watchParent () {
        const $parent = process.env.NODE_ENV !== 'production' ? '$parent.$parent' : '$parent.nodeInfo'
        this.$watch(`${$parent}.style.height`, () => this.$nextTick(() => {
          this.height = this.$el.clientHeight + 'px'
        }), { immediate: true })
        this.$watch(`${$parent}.child`, (val = []) => {
          this.slotslen = val.length
          this.pageMap = val
            .map(({id = 'noop'} = {}, i) => ([id, i]))
            .reduce((o, [id, i]) => Object.assign(o, {[id]: i}), {})
        }, { immediate: true })
        window.EMA && window.EMA.bind('select.one', (id) => {
          const $carousel = this.$refs.carousel || { setActiveItem: () => {} }
          const index = this.pageMap[id]
          if (index >= 0) $carousel.setActiveItem(index)
        })
      },
      change (index, old) {
        if (!inClient) console.log('page change', index, old)
        this.oncallExecute(this.onchange, [index, old])
      },
      prev () {
        this.$refs.carousel && this.$refs.carousel.prev()
      },
      next () {
        this.$refs.carousel && this.$refs.carousel.next()
      },
      setActiveItem (index) {
        this.$refs.carousel && this.$refs.carousel.setActiveItem(index)
      }
    }
  }
</script>

<style lang="stylus" rel="stylesheet/stylus" type="text/stylus" scoped>
  .component.carousel {
    width: 100%;
    height: 100%;
    >>> .el-carousel__arrow > .el-icon-arrow-right,
    >>> .el-carousel__arrow > .el-icon-arrow-left {
      &::before {
        content none 
      }
      &.el-icon-arrow-left {
        transform rotate(180deg)
      }
      background-image url('data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACAAAAAgCAYAAABzenr0AAABG0lEQVRYR+3TMU7DMBQG4N8vCyMjIxILCxOwIiY2xMbMxAkie6Sd4rz4DnQs7dqdC8AtuARIiZGlVqpQkfpsq1mc2fH/+fezwsifGjkfBVAaKA2IG2DmM+/9tzHmK8cTFgGaprmtqup9HfyotV6mIlIAITsZIQKExLZtJ0qpl62TJyHEgNyIKEBORDQgFyIJsAtBRFd1XX/u+zqSAdbaCyJaADhfh95rrVcHATjnrvu+XyilTkOg935qjJnsGx7WRTdgrb0hojcAJ7Hh0QBmvgMQwo9TwqMAXdc9eO/nAI5Sw8UA59zlMAwfmzuOufO/8yGaAWZ+AvCa4+QbiAgQfmLmZwA/WuuZZNr/WysG5Ajd3qMASgOlgdEb+AVCam4hbyGwtgAAAABJRU5ErkJggg==')
      background-size: 25px 25px;
      background-position: 50% center;
      display: inline-block;
      height: 36px;
      width: 36px;
      background-repeat: no-repeat;
    }
    >>> .el-carousel__indicators .el-carousel__button {
      box-shadow 0 0 20px 0px #888888
    }
  }
</style>
