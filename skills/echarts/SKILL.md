---
name: echarts
description:
  Create powerful interactive charts with Apache ECharts - balanced ease-of-use
  and customization. Use for broad chart variety, TypeScript support, mobile
  responsiveness, large datasets, and internationalized visualizations.
license: MIT
metadata:
  author: workspace-hub
  version: '1.0.0'
  category: data-visualization
  tags: [charts, echarts, apache, interactive, typescript, mobile]
  platforms: [web, javascript, typescript]
---

# Apache ECharts Visualization Skill

Create stunning, interactive charts with Apache ECharts - the perfect balance
of ease-of-use and extensive customization.

## When to Use This Skill

Use ECharts when you need:

- **Balance of ease and power** - Easy to start, powerful when needed
- **Broad chart variety** - 20+ chart types including geo maps
- **TypeScript support** - Full type definitions
- **Mobile responsiveness** - Built-in responsive design
- **Large datasets** - Efficient rendering of 100k+ points
- **Chinese/International** - Excellent i18n support

Avoid it when ultimate customization is needed (use D3.js), only simple charts
are needed (use Chart.js), or the project needs 3D scientific visualizations
(use Plotly).

## Core Capabilities

### 1. Basic Line Chart

```html
<!DOCTYPE html>
<html>
<head>
  <script src="https://cdn.jsdelivr.net/npm/echarts@5/dist/echarts.min.js"></script>
</head>
<body>
  <div id="main" style="width: 600px; height: 400px;"></div>
  <script>
    var myChart = echarts.init(document.getElementById('main'));
    var option = {
      title: { text: 'Monthly Sales' },
      tooltip: { trigger: 'axis' },
      legend: { data: ['Sales'] },
      xAxis: { type: 'category', data: ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun'] },
      yAxis: { type: 'value' },
      series: [{ name: 'Sales', type: 'line', data: [120, 200, 150, 80, 70, 110], smooth: true }]
    };
    myChart.setOption(option);
  </script>
</body>
</html>
```

### 2. Bar Chart with Multiple Series

```javascript
const option = {
  title: { text: 'Quarterly Revenue Comparison' },
  tooltip: { trigger: 'axis', axisPointer: { type: 'shadow' } },
  legend: { data: ['2023', '2024'] },
  grid: { left: '3%', right: '4%', bottom: '3%', containLabel: true },
  xAxis: { type: 'category', data: ['Q1', 'Q2', 'Q3', 'Q4'] },
  yAxis: { type: 'value', name: 'Revenue (k$)' },
  series: [
    { name: '2023', type: 'bar', data: [120, 200, 150, 80], itemStyle: { color: '#5470C6' } },
    { name: '2024', type: 'bar', data: [180, 250, 200, 120], itemStyle: { color: '#91CC75' } }
  ]
};
myChart.setOption(option);
```

### 3. Pie Chart with Rich Formatting

```javascript
const option = {
  title: { text: 'Traffic Sources', left: 'center' },
  tooltip: { trigger: 'item', formatter: '{a} <br/>{b}: {c} ({d}%)' },
  legend: { orient: 'vertical', left: 'left' },
  series: [{
    name: 'Access From', type: 'pie', radius: '50%',
    data: [
      { value: 1048, name: 'Search Engine' }, { value: 735, name: 'Direct' },
      { value: 580, name: 'Email' }, { value: 484, name: 'Affiliate' },
      { value: 300, name: 'Video Ads' }
    ],
    emphasis: { itemStyle: { shadowBlur: 10, shadowOffsetX: 0, shadowColor: 'rgba(0, 0, 0, 0.5)' } }
  }]
};
myChart.setOption(option);
```

## Complete Examples

### Loading Data from CSV

```javascript
fetch('../data/sales.csv')
  .then(response => response.text())
  .then(csvText => {
    const lines = csvText.trim().split('\n');
    const categories = [];
    const values = [];
    for (let i = 1; i < lines.length; i++) {
      const row = lines[i].split(',');
      categories.push(row[0]);
      values.push(parseFloat(row[1]));
    }
    const myChart = echarts.init(document.getElementById('main'));
    myChart.setOption({
      title: { text: 'Sales Data from CSV' }, tooltip: { trigger: 'axis' },
      xAxis: { type: 'category', data: categories }, yAxis: { type: 'value' },
      series: [{ name: 'Sales', type: 'line', data: values, smooth: true, areaStyle: {} }]
    });
  });
```

### Multi-Axis Chart

```javascript
const option = {
  title: { text: 'Temperature and Precipitation' },
  tooltip: { trigger: 'axis', axisPointer: { type: 'cross' } },
  legend: { data: ['Temperature', 'Precipitation'] },
  xAxis: { type: 'category', data: ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun'] },
  yAxis: [
    { type: 'value', name: 'Temperature (°C)', position: 'left', axisLabel: { formatter: '{value} °C' } },
    { type: 'value', name: 'Precipitation (mm)', position: 'right', axisLabel: { formatter: '{value} mm' } }
  ],
  series: [
    { name: 'Temperature', type: 'line', yAxisIndex: 0, data: [2, 5, 9, 15, 20, 25], smooth: true },
    { name: 'Precipitation', type: 'bar', yAxisIndex: 1, data: [50, 45, 40, 35, 30, 25] }
  ]
};
myChart.setOption(option);
```

### Heatmap Calendar

```javascript
function getVirtualData(year) {
  const date = +echarts.time.parse(`${year}-01-01`);
  const end = +echarts.time.parse(`${+year + 1}-01-01`);
  const data = [];
  for (let time = date; time < end; time += 3600 * 24 * 1000) {
    data.push([echarts.time.format(time, '{yyyy}-{MM}-{dd}', false), Math.floor(Math.random() * 10000)]);
  }
  return data;
}

myChart.setOption({
  title: { text: 'Activity Heatmap Calendar' },
  tooltip: { position: 'top', formatter: p => `${p.data[0]}: ${p.data[1]}` },
  visualMap: { min: 0, max: 10000, calculable: true, orient: 'horizontal', left: 'center', top: 'top' },
  calendar: { range: '2024', cellSize: ['auto', 13] },
  series: { type: 'heatmap', coordinateSystem: 'calendar', data: getVirtualData('2024') }
});
```

### Gauge Chart

```javascript
const option = {
  title: { text: 'Performance Score' },
  tooltip: { formatter: '{a} <br/>{b} : {c}%' },
  series: [{ name: 'Score', type: 'gauge', progress: { show: true },
    detail: { valueAnimation: true, formatter: '{value}%' },
    data: [{ value: 85, name: 'Overall Score' }] }]
};
myChart.setOption(option);
```

### Geographic Map

```javascript
fetch('https://cdn.jsdelivr.net/npm/echarts/map/json/china.json')
  .then(response => response.json())
  .then(chinaJson => {
    echarts.registerMap('china', chinaJson);
    myChart.setOption({
      title: { text: 'Sales by Province', left: 'center' },
      tooltip: { trigger: 'item', formatter: '{b}<br/>{c} (units)' },
      visualMap: { min: 0, max: 1000, text: ['High', 'Low'], calculable: true },
      series: [{ name: 'Sales', type: 'map', map: 'china', roam: true,
        emphasis: { label: { show: true } },
        data: [{ name: 'Beijing', value: 500 }, { name: 'Shanghai', value: 800 },
          { name: 'Guangdong', value: 900 }, { name: 'Zhejiang', value: 700 }] }]
    });
  });
```

### Dynamic Real-Time Data

```javascript
const data = [];
let now = new Date();
function randomData() {
  now = new Date(+now + 1000);
  return { name: now.toString(), value: [now.toLocaleString(), Math.round(Math.random() * 100)] };
}
for (let i = 0; i < 100; i++) data.push(randomData());
myChart.setOption({
  title: { text: 'Real-Time Data Stream' }, tooltip: { trigger: 'axis' },
  xAxis: { type: 'time', splitLine: { show: false } },
  yAxis: { type: 'value', boundaryGap: [0, '100%'], splitLine: { show: false } },
  series: [{ name: 'Value', type: 'line', showSymbol: false, data, smooth: true }]
});
setInterval(() => { data.shift(); data.push(randomData()); myChart.setOption({ series: [{ data }] }); }, 1000);
```

## Best Practices

### Responsive Design

```javascript
window.addEventListener('resize', () => myChart.resize());
// Or: myChart.resize({ width: 800, height: 600 });
```

### Loading and Empty States

```javascript
myChart.showLoading();
fetch('../data/data.json')
  .then(response => response.json())
  .then(data => {
    myChart.hideLoading();
    myChart.setOption(data.length === 0
      ? { title: { text: 'No Data Available', left: 'center', top: 'center' } }
      : option);
  });
```

### Themes and Large Datasets

```javascript
const myChart = echarts.init(document.getElementById('main'), 'dark');
// For large series: sampling: 'lttb', progressive: 1000, progressiveThreshold: 3000
```

## Available Chart Types

- Basic: Line, Bar, Pie, Scatter, Candlestick
- Advanced: Radar, Heatmap, Tree, Treemap, Sunburst, Parallel, Sankey, Funnel, Gauge
- Maps: GeoMap, BMap (Baidu Maps), Google Maps
- 3D with the GL extension: 3D Bar, 3D Line, 3D Scatter, 3D Surface, Globe
- Graphs: Graph and force-directed Graph

## TypeScript Support

```typescript
import * as echarts from 'echarts';

const option: echarts.EChartsOption = {
  title: { text: 'TypeScript Example' },
  xAxis: { type: 'category', data: ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun'] },
  yAxis: { type: 'value' },
  series: [{ data: [120, 200, 150, 80, 70, 110, 130], type: 'line' }]
};
const chartDom = document.getElementById('main')!;
const myChart = echarts.init(chartDom);
myChart.setOption(option);
```

## Installation

### CDN

```html
<script src="https://cdn.jsdelivr.net/npm/echarts@5/dist/echarts.min.js"></script>
```

### NPM

```bash
npm install echarts
```

```javascript
import * as echarts from 'echarts';
// Or use tree-shaking:
import * as echarts from 'echarts/core';
import { LineChart } from 'echarts/charts';
import { GridComponent } from 'echarts/components';
import { CanvasRenderer } from 'echarts/renderers';
echarts.use([LineChart, GridComponent, CanvasRenderer]);
```

## Advanced Features

```javascript
// Animation
const animated = { animation: true, animationDuration: 1000, animationEasing: 'cubicOut',
  animationDelay: idx => idx * 50 };

// Zoom and pan
const zoomable = { dataZoom: [{ type: 'inside', start: 0, end: 100 }, { type: 'slider', start: 0, end: 100 }] };

// Brush selection
myChart.setOption({ brush: { toolbox: ['rect', 'polygon', 'lineX', 'lineY', 'keep', 'clear'], xAxisIndex: 0 } });
myChart.on('brushSelected', params => console.log('Selected:', params.batch[0].selected[0].dataIndex));
```

## Event Handling

```javascript
myChart.on('click', params => alert('You clicked on ' + params.name));
myChart.on('mouseover', params => console.log('Hovered:', params));
myChart.dispatchAction({ type: 'highlight', seriesIndex: 0, dataIndex: 1 });
```

## Integration Examples

### React

```jsx
import { useEffect, useRef } from 'react';
import * as echarts from 'echarts';

function EChartsComponent({ option }) {
  const chartRef = useRef(null);
  const chartInstance = useRef(null);
  useEffect(() => {
    if (!chartInstance.current) chartInstance.current = echarts.init(chartRef.current);
    chartInstance.current.setOption(option);
    const handleResize = () => chartInstance.current.resize();
    window.addEventListener('resize', handleResize);
    return () => { window.removeEventListener('resize', handleResize); chartInstance.current?.dispose(); };
  }, [option]);
  return <div ref={chartRef} style={{ width: '100%', height: '400px' }} />;
}
```

### Vue

```vue
<template><div ref="chart" style="width: 600px; height: 400px;"></div></template>
<script>
import * as echarts from 'echarts';
export default {
  props: ['option'],
  mounted() { this.chart = echarts.init(this.$refs.chart); this.chart.setOption(this.option); window.addEventListener('resize', this.handleResize); },
  beforeUnmount() { window.removeEventListener('resize', this.handleResize); this.chart.dispose(); },
  methods: { handleResize() { this.chart.resize(); } },
  watch: { option: { deep: true, handler(value) { this.chart.setOption(value); } } }
};
</script>
```

## Resources

- Official Docs: https://echarts.apache.org/en/index.html
- Examples: https://echarts.apache.org/examples/en/index.html
- GitHub: https://github.com/apache/echarts
- Community: https://github.com/ecomfe/awesome-echarts

## Performance Tips

1. Use progressive rendering for more than 10k points.
2. Enable sampling for time-series data.
3. Lazy-load chart instances.
4. Dispose charts when unmounting.
5. Use the Canvas renderer for large datasets.

Use this skill for the best balance of ease-of-use and powerful customization.
