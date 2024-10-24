<template>
  <div class="full-width full-height box-border pa-4" :class="themeStyle==='black'? 'theme-black': ''">
    <div id="toolbar" class="row items-center mb-6">
      <slot name="topFilterLeft"></slot>
      <a-input-search
        v-model:value="keyword"
        style="width: 500px"
        placeholder="请输入关键字定位节点"
        @search="onSearch"
        class="ml-4"
      />
      <a-button class="ml-4" @click="onReset">重置</a-button>
      <a-radio-group v-model:value="view" class="ml-12" style="width: 230px" v-if="showSwitchView">
        <a-radio-button value="chart" style="width: 75px">火焰图</a-radio-button>
        <a-radio-button value="both" style="width: 75px">全部</a-radio-button>
        <a-radio-button value="table" style="width: 75px">表格</a-radio-button>
      </a-radio-group>
    </div>
    <div v-if="this.loading" class="full-width full-height row items-center">
      <a-spin class="full-width pt-100" :spinning="loading" size="large"></a-spin>
    </div>
    <div class="row" v-show="!this.loading">
      <div style="border-top: solid 1px #e8e8e8" ref="chartDiv">
        <div class="text-faded font-12 mb-1" v-if="view != viewType.table.value">
          说明：方块宽度越宽代表消耗越高，瓶颈问题通过宽度来显现。方块颜色区分不同的类库，相同颜色的方块代表堆栈属于同一个类库
        </div>
        <div :id="chartId" class="pb-15"></div>
      </div>
      <slot name="rightFilter" v-if="view != viewType.chart.value"></slot>
    </div>
  </div>
</template>
<script>
import {flamegraph} from 'd3-flame-graph'
import {defaultFlamegraphTooltip} from 'd3-flame-graph/src/tooltip'
import {mapGetters} from "vuex"

const viewType = {
  chart: {label: '火焰图', value: 'chart'},
  both: {label: '全部', value: 'both'},
  table: {label: '表格', value: 'table'}
}

export default {
  name: 'compFlameGraph',
  props: {
    chartData: {
      type: Object,
      default: {}
    },
    cellHeight: {
      type: Number,
      default: 20
    },
    width: {
      type: Number,
      default: 950
    },
    maxWidth: {
      type: Number,
      default: 1300
    },
    inverted: {
      // 火焰图翻转--root在上方，默认root在最下方
      type: Boolean,
      default: true
    },
    loading: {
      type: Boolean,
      default: false
    },
    showSwitchView: {
      type: Boolean,
      default: true
    },
    chartId: {
      type: String,
      default: 'chart'
    }
  },
  data: () => ({
    chart: null,
    keyword: null,
    viewType,
    view: 'both',
    chartWidth: 0
  }),
  computed: {
    ...mapGetters({
      themeStyle: 'getTheme',
    }),
  },
  watch: {
    chartData: {
      deep: true,
      handler(newValue, oldValue) {
        if (newValue) {
          this.keyword = null
          this.chart.update(newValue)
        }
      },
    },
    view: {
      immediate: true,
      handler(newValue, oldValue) {
        this.handleChartWithChange()
      }
    },
    width(value) {
      this.handleChartWithChange()
    },
    keyword(value) {
      // 关键字清空的时候
      !value && this.setGraphColor()
    }
  },
  methods: {
    initFlameGraph() {
      // Example on how to use custom a tooltip.
      let tip = defaultFlamegraphTooltip()
        .html(d => `<div class="pa-4 bg-light"><div>包名：${d.data.package}</div><div>方法名：${d.data.method}</div><div>样本：${d.data.readableValue ? d.data.readableValue : d.data.value} (${d.data.percent})</div></div>`)
      // .text(d => "name: " + d.data.name + ", value: " + d.data.value);

      this.chart = flamegraph()
        .width(this.chartWidth)
        .cellHeight(this.cellHeight)
        // .setColorHue("warm")
        .tooltip(tip)
        .inverted(this.inverted);

      // Purple if highlighted, otherwise the original color
      this.setGraphColor()

      d3.select(`#${this.chartId}`)
        .datum(this.chartData)
        .call(this.chart);
    },
    setGraphColor() {
      let highLightColor = "#7bd572"
      let defaultColor = "#8f8f8f"
      let color = defaultColor

      // 如果是过滤模式，则把非高亮的node置灰；一般模式正常展示
      if (this.keyword) {
        this.chart.setColorMapper(function (d, originalColor) {
          return d.highlight ? highLightColor : defaultColor
        })
      } else {
        this.chart.setColorMapper(function (d, originalColor) {
          // 方案1
          // let palette = [
          //   [0x50e150, 30, 30, 30],
          //   [0x50bebe, 30, 30, 30],
          //   [0xe17d00, 30, 30, 0],
          //   [0xc8c83c, 30, 30, 10],
          //   [0xe15a5a, 30, 40, 40],
          //   [0x50e150, 30, 50, 50],
          //   [0x50bebe, 30, 60, 60],
          //   [0xe17d00, 30, 70, 30],
          //   [0xc8c83c, 30, 80, 80],
          //   [0xe15a5a, 30, 90, 90],
          //   [0xe15a5a, 30, 90, 90],
          // ];
          //
          // let p = palette[d.data.type % palette.length]
          //
          // if (p) {
          //   const v = Math.random();
          //   color = '#' + (p[0] + ((p[1] * v) << 16 | (p[2] * v) << 8 | (p[3] * v))).toString(16);
          // }


          // 方案2
          let colors = ['#3f91f7', '#8784c4', '#efd59f', '#bed9af', '#7cc3d8', '#d48e5d', '#8cd5b5', '#6095c9', '#74bda4', '#ca89ca', '#9197c6', '#8ad9ea']
          let p = d.data.type % colors.length
          color = colors[p]

          return d.highlight ? highLightColor : color
        })
      }
    },
    onSearch(value) {
      this.setGraphColor()
      this.chart.search(value)
      this.$emit('search', value)
    },
    onReset() {
      this.keyword = null
      // 延时解决重置第一次全灰问题
      setTimeout(() => {
        this.chart.update(this.chartData)
        this.chart.resetZoom()
      }, 0);
      this.$emit('reset')
    },
    handleChartWithChange() {
      if (this.view) {
        if (this.view == viewType.table.value) {
          this.chartWidth = 0
        } else if (this.view == viewType.chart.value) {
          this.chartWidth = this.maxWidth
        } else {
          this.chartWidth = this.width
        }

        if (this.chart) {
          this.chart.width(this.chartWidth)
          this.chart.resetZoom()
        }
      }
    }
  },
  mounted() {
    this.initFlameGraph()
  }
}
</script>
<style>
.d3-flame-graph-tip {
  border-radius: 3px;
  padding: 0px 2px;
  min-width: 200px;
  text-align: left;
  font-size: 12px;
  color: #333;
  z-index: 1000;
}

/* 火焰图cell样式 */
.d3-flame-graph-label {
  border: solid 1px white;
}

.d3-flame-graph-label:hover {
  opacity: 0.8;
  background: #7bd572;
}
</style>
