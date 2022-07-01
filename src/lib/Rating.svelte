<script lang="ts">
  import Stars from "./Stars/Stars.svelte";

  import { createEventDispatcher } from "svelte";

  const dispatch = createEventDispatcher();

  const changeStar = (e) => {
    dispatch("change", e.detail);
  };

  export let comparable;
  export let item;
  export let value = 0;
  export let readonly = false;
</script>

<tr>
  <th class="text-left pr-2">{comparable.name}</th>
  <td class="flex"><Stars {value} {readonly} on:change={changeStar} /></td>
</tr>
