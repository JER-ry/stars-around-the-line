<script>
  import { onMount } from "svelte"
  import { Stage, Layer, Line, Rect, Image } from "svelte-konva"
  import blueStar from "$lib/blueStar.svg"
  import redStar from "$lib/redStar.svg"

  let blueStarObj, redStarObj
  onMount(() => {
    let blue = document.createElement("img")
    blue.src = blueStar
    blue.onload = () => {
      blueStarObj = blue
    }
    let red = document.createElement("img")
    red.src = redStar
    red.onload = () => {
      redStarObj = red
    }
  })

  let isPaint = false
  let lineProps
  let icons0 = []
  let icons1 = []
  let state = 0 // 0: display icons0, 1: display icons1

  function distance(x1, y1, x2, y2) {
    return Math.sqrt((x2 - x1) ** 2 + (y2 - y1) ** 2)
  }

  function getIntermediatePoints(x1, y1, x2, y2, numPoints) {
    let points = []
    for (let i = 1; i <= numPoints; i++) {
      let t = i / (numPoints + 1)
      let x = x1 + t * (x2 - x1)
      let y = y1 + t * (y2 - y1)
      points.push(x, y)
    }
    return points
  }

  function getRandomPointOnSegment(x1, y1, x2, y2) {
    let t = Math.random()
    let x = x1 + t * (x2 - x1)
    let y = y1 + t * (y2 - y1)
    return { x, y }
  }

  function getRandomPointNearSegment(x1, y1, x2, y2, R) {
    let { x, y } = getRandomPointOnSegment(x1, y1, x2, y2)
    let angle = Math.random() * Math.PI * 2
    let distance = Math.random() * R
    let rx = x + distance * Math.cos(angle)
    let ry = y + distance * Math.sin(angle)
    return { x: rx, y: ry }
  }

  function processPoints(points, image, D = 20, R = 40) {
    let icons = []
    let usefulPoints = []
    let i = 0
    let j = 1
    while (2 * j + 1 < points.length) {
      let x1 = points[2 * i]
      let y1 = points[2 * i + 1]
      let x2 = points[2 * j]
      let y2 = points[2 * j + 1]
      let dist = distance(x1, y1, x2, y2)
      if (dist > D || 2 * j + 1 === points.length - 1) {
        usefulPoints.push(x1, y1)
        let numIntermediatePoints = Math.floor(dist / D) - 1
        if (numIntermediatePoints >= 1) {
          let intermediatePoints = getIntermediatePoints(x1, y1, x2, y2, numIntermediatePoints)
          usefulPoints.push(...intermediatePoints)
        }
        i = j
        j++
      } else {
        j++
      }
    }
    for (let k = 0; 2 * k + 3 < usefulPoints.length; k++) {
      let x1 = usefulPoints[2 * k]
      let y1 = usefulPoints[2 * k + 1]
      let x2 = usefulPoints[2 * k + 2]
      let y2 = usefulPoints[2 * k + 3]
      let { x, y } = getRandomPointNearSegment(x1, y1, x2, y2, R)
      icons.push({
        x,
        y,
        image,
        width: 10,
        height: 10,
        rotation: Math.random() * 360
      })
    }
    return icons
  }

  function handleMouseDown(e) {
    isPaint = true
    icons0 = []
    icons1 = []
    let pos = e.detail.target.getStage().getPointerPosition()
    lineProps = {
      points: [pos.x, pos.y, pos.x, pos.y],
      stroke: "#df4b26",
      strokeWidth: 5,
      globalCompositeOperation: "source-over",
      lineCap: "round",
      lineJoin: "round"
    }
  }

  function handleMouseMove(e) {
    if (isPaint) {
      let pos = e.detail.target.getStage().getPointerPosition()
      lineProps.points = [...lineProps.points, pos.x, pos.y]
    }
  }

  function handleMouseUp(e) {
    isPaint = false
    icons0 = processPoints(lineProps.points, blueStarObj)
    icons1 = processPoints(lineProps.points, redStarObj)
  }

  function handleMouseLeave(e) {
    if (isPaint) handleMouseUp(e)
  }
</script>

<Stage
  config={{ width: 500, height: 500 }}
  on:mousedown={handleMouseDown}
  on:mousemove={handleMouseMove}
  on:mouseup={handleMouseUp}
  on:mouseleave={handleMouseLeave}>
  <Layer>
    <Rect config={{ x: 0, y: 0, width: 500, height: 500, fill: "#eeeeee" }} />
    {#if isPaint}
      <Line config={{ ...lineProps }} />
    {/if}
    {#if state === 0}
      {#each icons0 as icon}
        <Image config={{ ...icon }} />
      {/each}
    {:else}
      {#each icons1 as icon}
        <Image config={{ ...icon }} />
      {/each}
    {/if}
  </Layer>
</Stage>

<div class="flex flex-row space-x-4">
  <button
    class="bg-gray-500 hover:bg-gray-600 active:bg-gray-700 text-white font-bold px-1"
    on:click={() => {
      state = state === 0 ? 1 : 0
    }}>Toggle</button>
  <p class="text-gray-600 font-bold py-1">Current state: {state}</p>
  {#if !lineProps}
    <p class="text-gray-400 font-bold py-1">(Draw a line on the canvas first!)</p>
  {:else}
    <p class="text-gray-400 font-bold py-1">(Maybe try drawing another now?)</p>
  {/if}
</div>
