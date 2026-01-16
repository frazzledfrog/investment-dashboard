<script>
  import { onMount } from 'svelte';
  import LineChart from './components/LineChart.svelte';

  let data = null;
  let selectedMeasure = 'cumulative_index'; // 'returns' or 'cumulative_index'
  let logScale = true;
  let chartDataDates = [];
  let chartDataDatasets = [];

  // Color palette for lines
  const colors = [
    '#e60049', '#0bb4ff', '#50e991', '#e6d800', '#9b19f5', '#ffa300', '#dc0ab4', '#b3d4ff', '#00bfa0'
  ];

  onMount(async () => {
    try {
      const response = await fetch('/data.json');
      data = await response.json();
      updateChartData();
    } catch (error) {
      console.error('Failed to load data:', error);
    }
  });

  function updateChartData() {
    if (!data) return;

    chartDataDates = data.dates;
    
    // Transform series object into datasets array
    chartDataDatasets = Object.keys(data.series).map((name, index) => {
      return {
        label: name,
        data: data.series[name][selectedMeasure],
        borderColor: colors[index % colors.length],
        backgroundColor: colors[index % colors.length] + '20', // transparent fill
        tension: 0.1,
        pointRadius: 0
      };
    });
  }

  // Reactive updates
  $: if (data && selectedMeasure) {
    updateChartData();
  }
</script>

<main>
  <header>
    <h1>Financial Performance Dashboard</h1>
    <p class="subtitle">Interactive Investment Analysis</p>
  </header>

  <div class="controls glass-panel">
    <div class="control-group">
      <label>Measure</label>
      <div class="toggle-group">
        <button 
          class:active={selectedMeasure === 'cumulative_index'} 
          on:click={() => selectedMeasure = 'cumulative_index'}
        >
          Cumulative Index
        </button>
        <button 
          class:active={selectedMeasure === 'returns'} 
          on:click={() => selectedMeasure = 'returns'}
        >
          Returns
        </button>
      </div>
    </div>

    <div class="control-group">
      <label>Scale</label>
      <div class="toggle-group">
        <button 
          class:active={logScale} 
          on:click={() => logScale = true}
        >
          Logarithmic
        </button>
        <button 
          class:active={!logScale} 
          on:click={() => logScale = false}
        >
          Linear
        </button>
      </div>
    </div>
  </div>

  <div class="chart-wrapper glass-panel">
    {#if data}
      <LineChart 
        title={selectedMeasure === 'cumulative_index' ? 'Cumulative Performance (Base 100)' : 'Historical Returns'}
        labels={chartDataDates}
        datasets={chartDataDatasets}
        logScale={logScale}
      />
    {:else}
      <p class="loading">Loading market data...</p>
    {/if}
  </div>
</main>

<style>
  :global(:root) {
    --bg-color: #0f172a;
    --text-color: #f1f5f9;
    --accent: #3b82f6;
    --glass-bg: rgba(30, 41, 59, 0.7);
    --glass-border: rgba(255, 255, 255, 0.1);
  }

  :global(body) {
    margin: 0;
    font-family: 'Inter', system-ui, -apple-system, sans-serif;
    background-color: var(--bg-color);
    color: var(--text-color);
    min-height: 100vh;
  }

  main {
    max-width: 1200px;
    margin: 0 auto;
    padding: 2rem;
  }

  header {
    margin-bottom: 2rem;
    text-align: center;
  }

  h1 {
    font-size: 2.5rem;
    font-weight: 700;
    margin: 0;
    background: linear-gradient(to right, #60a5fa, #c084fc);
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
  }

  .subtitle {
    color: #94a3b8;
    margin-top: 0.5rem;
  }

  .glass-panel {
    background: var(--glass-bg);
    backdrop-filter: blur(10px);
    border: 1px solid var(--glass-border);
    border-radius: 1rem;
    padding: 1.5rem;
    box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -1px rgba(0, 0, 0, 0.06);
  }

  .controls {
    display: flex;
    justify-content: center;
    gap: 3rem;
    margin-bottom: 2rem;
  }

  .control-group label {
    display: block;
    font-size: 0.875rem;
    color: #94a3b8;
    margin-bottom: 0.5rem;
    text-align: center;
  }

  .toggle-group {
    display: flex;
    background: rgba(0, 0, 0, 0.2);
    border-radius: 0.5rem;
    padding: 0.25rem;
  }

  button {
    background: transparent;
    border: none;
    color: #94a3b8;
    padding: 0.5rem 1rem;
    border-radius: 0.25rem;
    cursor: pointer;
    font-weight: 500;
    transition: all 0.2s;
  }

  button:hover {
    color: #fff;
  }

  button.active {
    background: var(--accent);
    color: white;
    box-shadow: 0 1px 3px rgba(0,0,0,0.1);
  }

  .chart-wrapper {
    min-height: 500px;
  }

  .loading {
    text-align: center;
    color: #64748b;
    margin-top: 2rem;
  }
</style>
