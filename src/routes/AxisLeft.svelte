<!-- Adapted from https://datavisualizationwithsvelte.com/basics/dual-axis -->
<script>
  let { yScale, margin } = $props();
</script>

{#if yScale}
  <g transform="translate({margin.left},0)">
    <line stroke="currentColor" y1={yScale.range()[0]} y2={yScale.range()[1]} />
    {#each yScale.domain() as tick}
      {#if tick !== 0}
        <line
          stroke="currentColor"
          x1={0}
          x2={-6}
          y1={yScale(tick) + yScale.bandwidth() / 2}
          y2={yScale(tick) + yScale.bandwidth() / 2}
        />
      {/if}

      <text
        fill="currentColor"
        text-anchor="end"
        font-size="18"
        dominant-baseline="middle"
        x={-9}
        y={yScale(tick) + yScale.bandwidth() / 2}
      >
        {tick}
      </text>
    {/each}
  </g>
{/if}
