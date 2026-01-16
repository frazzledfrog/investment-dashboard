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
      // Use relative path for GitHub Pages compatibility
      const response = await fetch('./data.json');
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
    --bg-color: #f8fafc; /* Light background */
    --text-color: #1e293b; /* Dark text */
    --accent: #2563eb;
    --glass-bg: rgba(255, 255, 255, 0.7);
    --glass-border: rgba(203, 213, 225, 0.5);
  }

  :global(body) {
    margin: 0;
    font-family: 'Inter', system-ui, -apple-system, sans-serif;
    background-color: var(--bg-color);
    color: var(--text-color);
    min-height: 100vh;
  }

  main {
    max-width: 1600px; /* Wider for desktop */
    margin: 0 auto;
    padding: 2rem;
    height: 90vh; /* Taking up most of the viewport height */
    display: flex;
    flex-direction: column;
  }

  header {
    margin-bottom: 2rem;
    text-align: center;
  }

  h1 {
    font-size: 3rem;
    font-weight: 800;
    margin: 0;
    background: linear-gradient(to right, #2563eb, #7c3aed); /* Darker gradient for light mode */
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
  }

  .subtitle {
    color: #64748b;
    margin-top: 0.5rem;
    font-size: 1.1rem;
  }

  .glass-panel {
    background: var(--glass-bg);
    backdrop-filter: blur(12px);
    border: 1px solid var(--glass-border);
    border-radius: 1.5rem; /* Softer corners */
    padding: 1.5rem;
    box-shadow: 0 10px 25px -5px rgba(0, 0, 0, 0.1), 0 8px 10px -6px rgba(0, 0, 0, 0.05); /* Stronger shadow for depth */
  }

  .controls {
    display: flex;
    justify-content: center;
    gap: 4rem;
    margin-bottom: 2rem;
  }

  .control-group label {
    display: block;
    font-size: 0.875rem;
    font-weight: 600;
    color: #64748b;
    margin-bottom: 0.5rem;
    text-align: center;
    text-transform: uppercase;
    letter-spacing: 0.05em;
  }

  .toggle-group {
    display: flex;
    background: #e2e8f0; /* Light grey track */
    border-radius: 0.75rem;
    padding: 0.25rem;
  }

  button {
    background: transparent;
    border: none;
    color: #64748b;
    padding: 0.6rem 1.5rem;
    border-radius: 0.5rem;
    cursor: pointer;
    font-weight: 600;
    transition: all 0.2s ease-in-out;
  }

  button:hover {
    color: #1e293b;
    background: rgba(255,255,255,0.5);
  }

  button.active {
    background: white;
    color: var(--accent);
    box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -1px rgba(0, 0, 0, 0.06);
  }

  .chart-wrapper {
    flex: 1; /* Take remaining height */
    min-height: 0; /* Flexbox trick to allow shrinking */
    padding: 2rem;
    position: relative;
  }

  .loading {
    text-align: center;
    color: #64748b;
    margin-top: 2rem;
  }
</style>
