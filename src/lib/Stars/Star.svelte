<script lang="ts">
  export let title = "star";
  export let filled = false;
  export let size = 18;
  export let starId;
</script>

<svg
  xmlns="http://www.w3.org/2000/svg"
  width={size}
  height={size}
  viewBox="0 0 51 48"
  class="star"
  on:mouseover
  on:mouseleave
  on:click
  data-starid={starId}
  let:class
>
  <title>{title}</title>
  <path
    class:filled
    data-starid={starId}
    class="starPath"
    d="m25,1 6,17h18l-14,11 5,17-15-10-15,10 5-17-14-11h18z"
  />
</svg>

<style>
  .starPath {
    fill: none;
    stroke: rgba(255, 255, 255, 0.75);
  }
  .filled {
    stroke: yellow;
    fill: yellow;
  }
</style>
