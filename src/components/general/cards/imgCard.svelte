<script lang="ts">
	export let title: string = 'A title';
	export let img: string = 'Some text';
	export let right: boolean = false;
</script>

<article class="relative flex justify-between my-12 2xl:w-3/5 sm:w-4/5 items-center h-96">
	{#if !right}
		<img class="w-2/6 shadow-2xl rounded-full" src={img} alt={`${title} logo`} />
	{/if}
	<div class="w-3/5 p-16 bg-gray-100 rounded-lg shadow-md h-96">
		<h2 class="text-4xl font-semibold mb-4 border-b border-gray-200 pb-3">{title}</h2>
		<div class="text-lg">
			<slot />
		</div>
	</div>
	{#if right}
		<img class="w-2/6 shadow-2xl rounded-full" src={img} alt={`${title} logo`} />
	{/if}
</article>
