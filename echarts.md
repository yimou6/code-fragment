### ECharts的一些简单封装

可再echarts官网上编辑和下载echarts的主题文件

#### 导入和注册主题

___main.js___

```javascript
// 导入echarts和主题
import echarts from 'echarts'
const light = require('./light.json')
const dark = require('./dark.json')
// 注册主题
echarts.registerTheme('light', light)
echarts.registerTheme('dark', dark)
```

#### 封装基础图表组件

- 根据全局主题(vuex)切换图表主题

- 根据浏览器宽度调整图表宽度

- `emit`图表的`click`事件

```vue
<template>
   <div style="width: 100%" :style="{ height }" ref="chart"></div>
</template>

<script>
import echarts from 'echarts'
import { debounce } from 'lodash'
export default {
  name: 'BaseChart',
  props: {
    // 图表区域高度
    height: {
      type: String,
      default: '400px'
    },
    // echarts 配置项
    options: {
      type: Object,
      default: () => ({})
    }
  },
  watch: {
    '$store.getters.theme': function () {
      this.$nextTick(() => {
        this.initCharts()
      })
    },
    options: {
      deep: true,
      handler: function () {
        this.$nextTick(() => {
          this.updateCharts(this.options)
        })
      }
    },
  },
  data() {
    return {
      chart: ''
    }
  },
  mounted() {
    this.__resize = debounce(() => {
      if (this.chart) {
        this.chart.resize()
      }
    }, 50)
    window.addEventListener('resize', this.__resize)
  },
  methods: {
    // 初始化图表
    initCharts() {
      if (this.chart) {
        this.chart.clear()
        this.chart.dispose()
      }
      this.chart = echarts.init(this.$refs.chart, this.$store.getters.theme)
      // emit 图表点击事件
      this.chart.on('click', (params) => {
        this.$emit('click', params)
      })
    },
    // 更新图表
    updateCharts(Option) {
      if (!this.chart) {
        this.initCharts()
      }
      this.chart.setOption(Option, true)
    }
  },
  beforeDestroy() {
    window.removeEventListener('resize', this.__resize)
  }
}
</script>
```
