<script lang="ts">
  import Star from "./Star.svelte";

  import { createEventDispatcher } from "svelte";

  const dispatch = createEventDispatcher();

  export let stars = [
    { id: 1, title: "One Star" },
    { id: 2, title: "Two Stars" },
    { id: 3, title: "Three Stars" },
    { id: 4, title: "Four Stars" },
    { id: 5, title: "Five Stars" },
  ];

  export let value = 0;
  export let readonly = false;

  let rating = null,
    hoverRating = false;

  const handleHover = (id) => () => {
      if (!readonly) hoverRating = id;
    },
    handleRate = (id) => () => {
      if (!readonly) dispatch("change", { value: id });
    };
</script>

{#each stars as star (star.id)}
  <Star
    filled={value
      ? value >= star.id
      : hoverRating
      ? hoverRating >= star.id
      : rating >= star.id}
    starId={star.id}
    on:mouseover={handleHover(star.id)}
    on:mouseleave={() => (hoverRating = null)}
    on:click={handleRate(star.id)}
  />
{/each}
