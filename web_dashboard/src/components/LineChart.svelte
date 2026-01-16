<script>
  import { onMount } from 'svelte';
  import Chart from 'chart.js/auto';

  export let title = 'Chart';
  export let labels = [];
  export let datasets = [];
  export let logScale = false;

  let canvas;
  let chart;

  $: if (chart) {
    chart.data.labels = labels;
    chart.data.datasets = datasets;
    chart.options.scales.y.type = logScale ? 'logarithmic' : 'linear';
    chart.update();
  }

  onMount(() => {
    const ctx = canvas.getContext('2d');
    chart = new Chart(ctx, {
      type: 'line',
      data: {
        labels: labels,
        datasets: datasets
      },
      options: {
        responsive: true,
        maintainAspectRatio: false,
        plugins: {
          title: {
            display: true,
            text: title,
            color: '#e0e0e0',
            font: { size: 16 }
          },
          legend: {
            labels: { color: '#cccccc' }
          }
        },
        scales: {
          x: {
            ticks: { color: '#aaaaaa' },
            grid: { color: '#444444' }
          },
          y: {
            type: logScale ? 'logarithmic' : 'linear',
            ticks: { color: '#aaaaaa' },
            grid: { color: '#444444' }
          }
        },
        interaction: {
            mode: 'index',
            intersect: false,
        }
      }
    });

    return () => {
      chart.destroy();
    };
  });
</script>

<div class="chart-container">
  <canvas bind:this={canvas}></canvas>
</div>

<style>
  .chart-container {
    position: relative;
    height: 100%;
    width: 100%;
    min-height: 400px;
  }
</style>
