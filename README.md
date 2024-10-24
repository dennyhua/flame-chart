# flame-chart
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
<img width="1319" alt="flame effect" src="https://github.com/user-attachments/assets/c57c5c48-3463-4aad-90a1-064ecdbaba44">
