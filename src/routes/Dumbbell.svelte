<!-- Rect.svelte -->
<script lang="ts">
  import { Tween } from "svelte/motion";
  import { cubicOut } from "svelte/easing";
  import { interpolate } from "d3-interpolate";

  let { d, yBase, midpoint, leftTarget, rightTarget, i, fill } = $props();

  const tweenParams = {
    duration: 250,
    easing: cubicOut,
    interpolate,
  };

  const yMid = yBase - 3;
  const yRect = yBase - 5;

  const isCommentLeft =
    d.CommentStudentizedScore < d.StatStudentizedScr ? true : false;

  let tXLeft = new Tween<number>(midpoint, {
    ...tweenParams,
    delay: 300 + i * 50,
  });

  let tXRight = new Tween<number>(midpoint, {
    ...tweenParams,
    delay: 300 + i * 50,
  });

  $effect(() => {
    tXLeft.target = leftTarget;
    tXRight.target = rightTarget;
  });
</script>

{#if isCommentLeft}
  <rect
    x={leftTarget}
    y={yMid}
    width={tXRight.current - tXLeft.current}
    height={5}
    {fill}
  />
  <circle cx={tXLeft.current} cy={yBase} r="5" {fill} />
  <rect x={tXRight.current} y={yRect} width="10" height="10" {fill} />
{/if}
{#if !isCommentLeft}
  <rect
    x={leftTarget}
    y={yMid}
    width={tXRight.current - tXLeft.current}
    height={5}
    {fill}
  />
  <circle cx={tXRight.current} cy={yBase} r="5" {fill} />
  <rect x={tXLeft.current} y={yRect} width="10" height="10" {fill} />
{/if}
