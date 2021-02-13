<script>
  import { LayerCake, Svg, Html, flatten } from 'layercake';
  import { stack } from 'd3-shape';
  import { scaleOrdinal } from 'd3-scale';
  import { format, precisionFixed } from 'd3-format';
  import { timeParse, timeFormat } from 'd3-time-format';

  import AxisX from './components/AxisX.svelte';
  import AxisY from './components/AxisY.svelte';
  import AreaStacked from './components/AreaStacked.svelte';
  import SharedTooltip from './components/SharedTooltip.svelte';

  // This example loads csv data as json using @rollup/plugin-dsv
  import data from './data/fruit.csv';

  const xKey = 'month';
  const yKey = [0, 1];
  const zKey = 'key';

  const parseDate = timeParse('%Y-%m-%d');

  const seriesNames = Object.keys(data[0]).filter(d => d !== xKey);
  const seriesColors = ['#ff00cc', '#ff7ac7', '#ffb3c0', '#ffe4b8'];

  data.forEach(d => {
    d[xKey] = typeof d[xKey] === 'string' ? parseDate(d[xKey]) : d[xKey];
    seriesNames.forEach(name => {
      d[name] = +d[name];
    });
  });

  /* --------------------------------------------
   * Create a stacked data structure
   */
  const stackData = stack()
    .keys(seriesNames);

  const series = stackData(data);

  const formatTickX = timeFormat('%b. %-d')
  const formatTickY = d => format(`.${precisionFixed(d)}s`)(d);
</script>

<style>
  .chart-container {
    width: 100%;
    height: 250px;
  }
</style>

<div class="chart-container">
  <LayerCake
    padding={{ top: 0, right: 0, bottom: 20, left: 17 }}
    x={d => {
      console.log('d', d);
      console.log('d data', d.data);
      return d.data[xKey];
    }}
    y={yKey}
    z={zKey}
    zScale={scaleOrdinal()}
    zDomain={seriesNames}
    zRange={seriesColors}
    flatData={flatten(series)}
    data={series}
  >
    <Svg>
      <AxisX
        snapTicks={true}
        ticks={data.map(d => d.month).reverse()}
        formatTick={formatTickX}
      />
      <AxisY
        formatTick={formatTickY}
      />
      <AreaStacked/>
    </Svg>

    <Html>
      <SharedTooltip
        formatTitle={formatTickX}
        dataset={data}
      />
    </Html>
  </LayerCake>
</div>
