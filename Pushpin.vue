<!--params
@prop position 停靠的位置 JSON 默认：{top: 0, left: 0}
@prop scrollElement 监听滚动的元素 String 默认：document
@prop advance 激进策略，采用position: sticky; Boolean 默认：false
@prop onFixed isFixed=true时触发的钩子函数，此事件对advance=true时无效, 函数参数（{position: Object, clientRect: Object}）
@prop onFixDisabled isFixed=false时触发的钩子函数，此事件对advance=true时无效, 函数参数（{position: Object, clientRect: Object}）
@slot inner 填入图钉的内容 默认：空
-->
<template>
    <div  :class="['lm-pushpin', {'lm-pushpin-advance': advance}]" 
          :style="position">
      <div :class="['lm-pushpin-inner', {'lm-pushpin-inner-fixed': isFixed}]" 
           :style="position">
        <slot name="inner"></slot>
      </div>
      <div class="lm-pushpin-placeholder" :style="placeholderStyle"></div>
    </div>
</template>
<style lang="scss">
.lm-pushpin-inner-fixed {
  position: fixed;
  z-index: 1000;
}
.lm-pushpin-advance {
  position: -webkit-sticky;
  position: sticky;
  z-index: 1000;
}
</style>
<script>
  export default{
    props: {
        scrollElement: {
          type: String,
          default: 'null'
        },
        position: Object,
        advance: {
          type: Boolean,
          default: false
        },
        onFixed: Function,
        onFixDisabled: Function
    },
    data () {
      return {
        placeholderStyle: { // 占位符的样式
          height: 0,
          width: 0
        },
        isFixed: false,
        scrollNode: Object
      }
    },
    methods: {
      onScroll: function () {
        if (!this.advance) {
          if (this.$el.getBoundingClientRect().top < parseInt(this.position.top) || this.$el.getBoundingClientRect().left < parseInt(this.position.left) || (document.documentElement.clientWidth - this.$el.getBoundingClientRect().right) < parseInt(this.position.right) || (document.documentElement.clientHeight - this.$el.getBoundingClientRect().bottom) < parseInt(this.position.bottom)) {
            this.isFixed = true
          } else {
            this.isFixed = false
          }
        }
      }
    },
    components: {
    },
    mounted: function () {
      if (this.scrollElement === 'document') {
        this.scrollNode = document
      } else if (this.scrollElement === 'window') {
        this.scrollNode = window
      } else if (this.scrollElement === 'null') { // 自动判断节点overflow来决定挂载哪个节点
        console.log(this.$el.parentNode.style['overflow'])
        console.log(document.documentElement.style['overflow'])
        if (this.$el.parentNode.style['overflow'] === 'scroll') {
          this.scrollNode = this.$el.parentNode
        } else {
          this.scrollNode = document
        }
      } else if (this.scrollElement === 'parent') {
        this.scrollNode = this.$el.parentNode
      } else {
        this.scrollNode = document.querySelector(this.scrollElement)
      }
      this.scrollNode.addEventListener('scroll', this.onScroll) // 绑定滚动事件
      // 初始化时检测是否已经可以浮动
      if (this.$el.getBoundingClientRect().top < parseInt(this.position.top) || this.$el.getBoundingClientRect().left < parseInt(this.position.left) || (document.documentElement.clientWidth - this.$el.getBoundingClientRect().right) < parseInt(this.position.right) || (document.documentElement.clientHeight - this.$el.getBoundingClientRect().bottom) < parseInt(this.position.bottom)) {
        this.isFixed = true
      } else {
        this.isFixed = false
      }
    },
    watch: {
      isFixed: function (val) {
        if (val) {
          this.placeholderStyle.height = this.$el.getElementsByClassName('lm-pushpin-inner')[0].clientHeight + 'px'
          this.placeholderStyle.width = this.$el.getElementsByClassName('lm-pushpin-inner')[0].clientWidth + 'px'
          // isFixed = true 时触发
          if (this.onFixed !== undefined) {
            this.onFixed(this.position, this.$el.getBoundingClientRect())
          }
        } else {
          this.placeholderStyle.height = '0'
          this.placeholderStyle.width = '0'
          // isFixed = false 时触发
          if (this.onFixDisabled !== undefined) {
            this.onFixDisabled(this.position, this.$el.getBoundingClientRect())
          }
        }
      }
    },
    computed: {
    }
  }
</script>
