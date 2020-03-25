<script>
import { Line } from 'vue-chartjs';

export default {
  extends: Line,
  name: 'Chart',
  props: ['chartdata', 'options', 'home'],
  mounted() {
    this.renderChart(this.chartdata, this.options);
  },
  watch: {
    chartdata: function() {
      // this._data._chart.update();
      this.renderChart(this.chartdata, this.options);
    },
  },
};
</script>
