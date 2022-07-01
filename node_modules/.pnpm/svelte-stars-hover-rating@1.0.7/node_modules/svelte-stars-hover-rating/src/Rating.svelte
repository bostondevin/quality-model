<script>
	import Star from './Star.svelte';
	
    export let rating=null
	export let num = 5
	let hoverRating = null;

	const handleHover = (id) => () => {
		hoverRating = id;
	}
	const handleRate = (id) => () => {
		rating = id;
	}
	let stars = Array.from({length:num},(__,i)=>i+1)
</script>
<style>
	.feedback {
		position: relative;
	}

	.starContainer {
		display: inline-block;
		border-radius: 8px;
		padding: 4px 6px 0;
	}

	:global(button) {
		cursor: pointer;
	}
</style>

<div class="feedback"> 
		<span class="starContainer">
		{#each stars as star}
			<Star 
					filled={hoverRating ? (hoverRating >= star) : (rating >= star)} 
					starId={star}
					on:mouseover={handleHover(star)} 
					on:mouseleave={() => hoverRating = null}
					on:click={handleRate(star)}
				/>
		{/each}

  </span>
</div>