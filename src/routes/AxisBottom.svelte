<script lang="ts">
  // Props
  let { xScale, margin, height, width } = $props();
</script>

{#if xScale}
  <g transform="translate(0,{height - margin.bottom})">
    <!-- Axis line -->
    <line stroke="currentColor" x1={margin.left} x2={width - margin.right} />

    <!-- All ticks -->
    {#each xScale.domain() as tick}
      <line
        stroke="currentColor"
        x1={xScale(tick) + xScale.bandwidth() / 2}
        x2={xScale(tick) + xScale.bandwidth() / 2}
        y1={0}
        y2={6}
      />

      <text
        font-size="16px"
        fill="currentColor"
        text-anchor="middle"
        x={xScale(tick) + xScale.bandwidth() / 2}
        y={20}
      >
        {tick}
      </text>
    {/each}
  </g>
{/if}
