<script lang="ts">
  import { csv } from "d3-fetch";
  import { scaleBand, scaleLinear, scaleOrdinal } from "d3-scale";
  import { onMount } from "svelte";
  import { min, max } from "d3-array";
  import { quadtree } from "d3-quadtree";
  import AxisBottom from "./AxisBottom.svelte";
  import AxisLeft from "./AxisLeft.svelte";
  import type { NbaData } from "$lib/types";
  import type { Quadtree } from "d3-quadtree";
  import { Tween } from "svelte/motion";
  import { cubicOut } from "svelte/easing";
  import { interpolate } from "d3-interpolate";
  import Dumbbell from "./Dumbbell.svelte";

  // Import data
  let data: any[] = $state([]);
  onMount(() => {
    csv("/data/comment_stat_comparison.csv")
      .then((csvData) => {
        data = csvData;
      })
      .catch((e) => {
        console.log(e);
      });
  });

  // Set dimensions, margins, and scales
  let width = $state(800);
  const height = 800;
  const margin = { top: 30, right: 30, bottom: 50, left: 200 };
  const x_ticks = [-3, -2, -1, 0, 1, 2, 3];

  let x = $derived(
    data && width
      ? scaleLinear()
          .domain([min(x_ticks)!, max(x_ticks)!])
          .range([margin.left, width - margin.right])
      : null,
  );

  let y = $derived(
    data
      ? scaleBand()
          .domain(data.map((d) => d.Player))
          .range([height - margin.bottom, margin.top])
          .padding(0.1)
      : null,
  );

  let xScale = $derived(
    data && width
      ? scaleBand()
          .domain(x_ticks.map((x) => String(x)))
          .range([margin.left, width - margin.right])
      : null,
  );

  let yScale = $derived(
    data && width
      ? scaleBand()
          .domain(data.map((d) => d.Player))
          .range([height - margin.bottom, margin.top])
      : null,
  );

  // Handle hover for more info
  let finder: Quadtree<NbaData> | null = $state(null);
  let svgElement: SVGSVGElement | null = $state(null);
  let hoveredElem: NbaData | undefined = $state(undefined);
  let hoverX: number | null = $state(null);
  let hoverY: number | null = $state(null);

  $effect(() => {
    if (data && xScale && yScale && x && y) {
      finder = quadtree<NbaData>()
        .x((d) =>
          x(
            Math.abs(d.Difference) / 2 +
              Math.min(d.CommentStudentizedScore, d.StatStudentizedScr),
          ),
        )
        .y((d) => y(d.Player)! + y.bandwidth() / 2)
        .addAll(data);
    }
  });

  function getSvgCoords(event: MouseEvent, svg: SVGSVGElement) {
    const pt = svg.createSVGPoint();
    pt.x = event.clientX;
    pt.y = event.clientY;
    const svgP = pt.matrixTransform(svg.getScreenCTM()?.inverse());
    return { svgX: svgP.x, svgY: svgP.y };
  }

  function svgPointToPage(svg: SVGSVGElement, xSvg: number, ySvg: number) {
    const pt = svg.createSVGPoint();
    pt.x = xSvg;
    pt.y = ySvg;
    const pagePt = pt.matrixTransform(svg.getScreenCTM()!);
    return { pageX: pagePt.x, pageY: pagePt.y };
  }

  function handleMouseMove(event: MouseEvent) {
    if (finder && svgElement && x && y) {
      const { svgX, svgY } = getSvgCoords(event, svgElement);
      const d = finder.find(svgX, svgY);
      hoveredElem = d;
      const { pageX, pageY } = svgPointToPage(
        svgElement,
        x(
          Math.abs(hoveredElem?.Difference ?? 0) / 2 +
            Math.min(
              hoveredElem?.CommentStudentizedScore ?? 0,
              hoveredElem?.StatStudentizedScr ?? 0,
            ),
        ),
        y(hoveredElem?.Player ?? "")! + y.bandwidth() / 2,
      );
      hoverX = pageX;
      hoverY = pageY;
    }
  }

  // Create legend
  const legend_data = ["Net Rating", "Comment Ratio", "Overhated", "Overloved"];
  const colors = scaleOrdinal()
    .range([
      "rgb(209, 209, 209)",
      "rgb(209, 209, 209)",
      "rgb(217, 98, 98)",
      "rgb(117, 209, 104)",
    ])
    .domain(legend_data);

  // Animate dumbbells
  const tweenParams = {
    duration: 250,
    easing: cubicOut,
    interpolate,
  };

  let tX = new Tween<number>(0, {
    ...tweenParams,
    delay: 200,
  });
</script>

<div class="wrapper">
  {#if data && width && x && y}
    <!-- <div class="label-wrapper">
      <div class="label-container">
        <div class="overhated side-label">Overhated</div>
      </div>
      <div class="label-container">
        <div class="overloved side-label">Overloved</div>
      </div>
    </div> -->
    <div aria-hidden="true" onmousemove={handleMouseMove}>
      {#if hoveredElem && hoverX && hoverY}
        <div class="tooltip" style="left:{hoverX}px; top:{hoverY}px">
          <div>
            <b>{hoveredElem.Player}</b><br />
            Net Rating: {Math.round(hoveredElem.StatStudentizedScr * 100) /
              100}<br />
            Comment Ratio: {Math.round(
              hoveredElem.CommentStudentizedScore * 100,
            ) / 100} <br />
            Difference: {Math.round(hoveredElem.Difference * 100) / 100}
          </div>
        </div>
      {/if}
      <svg bind:this={svgElement} {width} {height}>
        {#each data as d, i}
          <Dumbbell
            {d}
            yBase={y(d.Player)! + y.bandwidth() / 2}
            midpoint={x(
              Math.abs(d.Difference) / 2 +
                Math.min(d.CommentStudentizedScore, d.StatStudentizedScr),
            )}
            leftTarget={Math.min(
              x(d.StatStudentizedScr),
              x(d.CommentStudentizedScore),
            )}
            rightTarget={Math.max(
              x(d.StatStudentizedScr),
              x(d.CommentStudentizedScore),
            )}
            {i}
            fill={d.Difference < 0 ? "rgb(217, 98, 98)" : "rgb(117, 209, 104)"}
          />
        {/each}
        <AxisBottom
          {width}
          {height}
          {margin}
          {xScale}
          text={"Standard Deviations Away from the Mean"}
        />
        <AxisLeft {yScale} {margin} />
        <g transform="translate({width - margin.right},130)">
          <rect
            x="-115"
            y={-20}
            width="145"
            height="150"
            fill="rgb(235, 235, 235)"
            rx="10px"
          />
          <text x="-70" font-weight="bold"> Legend </text>
          {#each legend_data as d, i}
            {#if d !== "Comment Ratio"}
              <rect
                x="-105"
                y={i * 30 + 10}
                width="20"
                height="20"
                fill={colors(d) as string}
              />
            {/if}
            {#if d == "Comment Ratio"}
              <circle
                cx="-95"
                cy={i * 30 + 20}
                r="10"
                fill={colors(d) as string}
              />
            {/if}
            <text class="legend-text" x="-80" y={25 + i * 30}>
              {d}
            </text>
          {/each}
        </g>
      </svg>
    </div>
  {/if}
</div>

<style>
  .label-wrapper {
    display: flex;
    flex-direction: column;
    justify-content: space-between;
    height: 100%;
    width: 36px;
  }

  .label-container {
    display: flex;
    height: 100%;
    justify-content: center;
    align-items: center;
  }

  .side-label {
    font-family: Arial, Helvetica, sans-serif;
    font-weight: bolder;
    -webkit-transform: rotate(-90deg);
  }

  .overhated {
    color: rgb(217, 98, 98);
  }

  .overloved {
    color: rgb(117, 209, 104);
  }

  .wrapper {
    width: 95%;
    height: 100%;
    margin: 0 auto;
    display: flex;
    justify-content: center;
    align-items: center;
  }

  .tooltip {
    position: absolute;
    font-family: "Poppins", sans-serif !important;
    min-width: 8em;
    line-height: 1.2;
    font-size: 0.875rem;
    z-index: 1;
    padding: 6px;
    transition:
      left 100ms ease,
      top 100ms ease;
    display: flex;
    justify-content: center;
    background: rgba(255, 255, 255, 0.51);
    border-radius: 5px;
    box-shadow: 0 4px 30px rgba(0, 0, 0, 0.1);
    backdrop-filter: blur(5px);
    -webkit-backdrop-filter: blur(5px);
    border: 1px solid rgba(255, 255, 255, 0.3);
  }
</style>
