<script>
  import { onMount } from "svelte";
  import Chart from "chart.js/auto";

  export let title = "Chart";
  export let labels = [];
  export let datasets = [];
  export let logScale = false;

  let canvas;
  let chart;

  $: if (chart) {
    chart.data.labels = labels;
    chart.data.datasets = datasets;
    chart.options.scales.y.type = logScale ? "logarithmic" : "linear";
    chart.update();
  }

  onMount(() => {
    const ctx = canvas.getContext("2d");
    chart = new Chart(ctx, {
      type: "line",
      data: {
        labels: labels,
        datasets: datasets,
      },
      options: {
        responsive: true,
        maintainAspectRatio: false,
        plugins: {
          title: {
            display: true,
            text: title,
            color: "#334155", // Slate-700
            font: { size: 16 },
          },
          legend: {
            labels: { color: "#475569" }, // Slate-600
          },
        },
        scales: {
          x: {
            ticks: { color: "#64748b" }, // Slate-500
            grid: { color: "#cbd5e1" }, // Slate-300
          },
          y: {
            type: logScale ? "logarithmic" : "linear",
            ticks: { color: "#64748b" },
            grid: { color: "#cbd5e1" },
          },
        },
        interaction: {
          mode: "index",
          intersect: false,
        },
      },
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
