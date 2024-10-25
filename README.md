# flame-chart 火焰图
flame chart component, 一款居于d3-flame-graph的火焰图，支持数据以表格、火焰图展示

# usage
使用方式
      <comp-flame-chart
        :chart-data="flameGraphData"
        :loading="loading"
        :width="900"
        :max-width="1300"
        v-if="flameGraphData != null"
        @reset="onReset"
        @search="onSearch"> </comp-flame-chart>

# 效果图
<img width="1315" alt="flame" src="https://github.com/user-attachments/assets/6c1b41b2-d635-4c9f-a8a9-ea50a831352a">
