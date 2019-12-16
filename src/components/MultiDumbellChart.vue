<template>
  <div
    :id='id'
    v-waypoint="{
      active: animationActive && !calcPercent,
      callback: appearOnScroll,
      options: intersectionOptions,
    }"
  />
</template>

<script>

export default {
  name: 'MultiDumbellChart',

  data() {
    return {
      chart: null,
      id: `chardiv_${Math.random().toString(30).substr(2, 8)}`,

      lines: null,
      points: [],

      rotated: false,
      appeared: false,

      intersectionOptions: {
        threshold: [0.5],
      },
    };
  },

  props: {
    data: { type: Array, required: true },
    values: { type: Array },

    horizontal: { type: Boolean, default: false },
    min: { type: Number },
    max: { type: Number },

    category: { type: String, default: 'category' },
    categoryTitle: { type: String },

    valueTitle: { type: String },
    valueFormat: { type: String, default: '#' },
    tooltipText: { type: String, default: '{categoryY}: {valueX}' },

    calcPercent: { type: Boolean, default: false },
    disableLegend: { type: Boolean, default: false },

    axisBreak: {
      type: Object,
      validator(axisBreak) { // expects { start: x, end: y, breakSize: z }
        if (!axisBreak) return true; // accept null

        // must be an object
        if (Object.prototype.toString.call(axisBreak) !== '[object Object]') return false;

        // must set limits and size
        if (!('start' in axisBreak) || !('end' in axisBreak) || !('breakSize' in axisBreak)) return false;

        return true;
      },
    },

    height: { type: String, default: '30rem' },
    rotationBreakpoint: { type: Number, default: 1000 },
    rotatedHeight: { type: String, default: '90vh' },

    colorOffset: { type: Number, default: -1 },
    animationDuration: { type: Number, default: 500 },
    animationActive: { type: Boolean, default: true },
  },

  methods: {
    drawChart() {
      const am4core = this.$am4core;
      const am4charts = this.$am4charts;

      // create chart
      const chart = this.$am4core.create(this.id, am4charts.XYChart);
      this.chart = chart;
      chart.data = this.data;

      // determine if the chart will be horizontal or vertical
      const windHeight = window.innerHeight || document.documentElement.clientHeight;
      const windWidth = window.innerWidth || document.documentElement.clientWidth;
      this.rotated = windWidth < windHeight && chart.pixelWidth < this.rotationBreakpoint;
      const horizontal = (this.horizontal && !this.rotated) || (!this.horizontal && this.rotated);

      if (this.rotated && this.rotatedHeight) {
        chart.svgContainer.htmlElement.style.height = this.rotatedHeight;
      } else {
        chart.svgContainer.htmlElement.style.height = this.height;
      }

      let categoryAxis;
      let valueAxis;

      if (horizontal) {
        categoryAxis = chart.yAxes.push(new am4charts.CategoryAxis());
        valueAxis = chart.xAxes.push(new am4charts.ValueAxis());
      } else {
        categoryAxis = chart.xAxes.push(new am4charts.CategoryAxis());
        valueAxis = chart.yAxes.push(new am4charts.ValueAxis());
      }

      categoryAxis.dataFields.category = this.category;
      categoryAxis.title.text = this.categoryTitle;
      categoryAxis.title.fontSize = '1rem';
      categoryAxis.fontSize = '.8rem';

      categoryAxis.renderer.line.strokeOpacity = 1;
      categoryAxis.renderer.grid.template.disabled = true;
      categoryAxis.renderer.minGridDistance = 1;
      categoryAxis.renderer.labels.template.truncate = false;
      categoryAxis.renderer.labels.template.hideOversized = false;
      categoryAxis.renderer.labels.template.verticalCenter = 'middle';

      if (horizontal) { // display top to bottom when bars are horizontal
        categoryAxis.renderer.inversed = true;
        valueAxis.renderer.minGridDistance = 100;
      } else { // rotate labels when bars are vertical
        categoryAxis.renderer.labels.template.rotation = 270;
        categoryAxis.renderer.labels.template.horizontalCenter = 'right';
        valueAxis.renderer.minGridDistance = 40;
      }

      valueAxis.title.text = this.valueTitle;
      valueAxis.title.fontSize = '1rem';
      valueAxis.fontSize = '.8rem';

      valueAxis.min = this.min;
      valueAxis.max = this.max;

      valueAxis.numberFormatter = new am4core.NumberFormatter();
      valueAxis.numberFormatter.numberFormat = this.valueFormat;

      // range lines
      const lines = chart.series.push(new am4charts.ColumnSeries());
      this.lines = lines;

      lines.dataFields.category = this.category;
      lines.dataFields.categoryX = this.category;
      lines.dataFields.categoryY = this.category;

      lines.dataFields.openValueY = this.values[0].value;
      lines.dataFields.openValueX = this.values[0].value;
      lines.dataFields.valueY = this.values[this.values.length - 1].value;
      lines.dataFields.valueX = this.values[this.values.length - 1].value;

      lines.columns.template.width = 0.01;
      lines.columns.template.height = 0.01;
      lines.fillOpacity = 0;
      lines.strokeOpacity = 1;

      lines.hidden = true;
      lines.hiddenInLegend = true;

      const duration = this.animationDuration / (this.data.length + 1);
      lines.interpolationDuration = duration * 2;
      lines.sequencedInterpolationDelay = duration;
      lines.sequencedInterpolation = true;

      // add points
      const self = this;
      this.points = [];

      function createPoints(value, name) {
        const points = chart.series.push(new am4charts.LineSeries());
        self.points.push(points);
        points.strokeOpacity = 0;

        points.name = name || value;
        points.dataFields.value = value;
        points.dataFields.valueX = value;
        points.dataFields.valueY = value;
        points.dataFields.category = self.category;
        points.dataFields.categoryX = self.category;
        points.dataFields.categoryY = self.category;

        const circles = points.bullets.push(new am4core.Circle());
        circles.radius = 5;

        const color = chart.colors.getIndex(chart.series.length - 1 + self.colorOffset);
        circles.fill = color;
        circles.stroke = color;

        // modify tooltip
        circles.tooltipText = self.tooltipText;

        points.tooltip.getFillFromObject = false;
        points.tooltip.background.fill = am4core.color('#fff');
        points.tooltip.label.fill = am4core.color('#000');
        points.tooltip.label.wrap = true;
        points.tooltip.label.maxWidth = windWidth * 0.8;
        points.tooltip.pointerOrientation = 'vertical';

        // animations
        points.interpolationDuration = duration * 2;
        points.sequencedInterpolationDelay = duration;
        points.sequencedInterpolation = true;

        // start hidden
        points.hidden = self.animationActive;
      }

      for (let i = 0; i < this.values.length; i += 1) {
        createPoints(this.values[i].value, this.values[i].name);
      }
      this.appeared = false;

      // Legend
      chart.legend = new am4charts.Legend();
      chart.legend.fontSize = '.8rem';
      chart.legend.position = 'top';

      // disable zoom out
      chart.zoomOutButton.disabled = true;

      // disable legend
      if (this.disableLegend) {
        chart.legend.itemContainers.template.clickable = false;
        chart.legend.itemContainers.template.focusable = false;
        chart.legend.itemContainers.template.cursorOverStyle = am4core.MouseCursorStyle.default;
      }
    },

    appearOnScroll({ going }) {
      const { points, lines, appeared } = this;

      if (!appeared && going === this.$waypointMap.GOING_IN) {
        lines.hidden = false;
        lines.appear();

        for (let i = 0; i < points.length; i += 1) {
          points[i].hidden = false;
          points[i].appear();
        }

        this.appeared = true;
      }
    },

    rotateOnResize() {
      const { chart, rotated, rotationBreakpoint } = this;
      const windHeight = window.innerHeight || document.documentElement.clientHeight;
      const windWidth = window.innerWidth || document.documentElement.clientWidth;

      if (windWidth < windHeight && chart.pixelWidth < rotationBreakpoint) {
        // rotate
        if (!rotated) {
          this.rotated = true;
          chart.dispose();
          this.drawChart();
        }
      } else if (rotated) {
        // return to normal orientation
        this.rotated = false;
        chart.dispose();
        this.drawChart();
      }
    },
  },

  mounted() {
    this.drawChart();
    window.addEventListener('resize', this.rotateOnResize, false);
  },

  beforeDestroy() {
    if (this.chart) {
      this.chart.dispose();
      window.removeEventListener('resize', this.rotateOnResize, false);
    }
  },
};
</script>

<style scoped></style>
