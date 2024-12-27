<script>
	export let choice;
	export let isSelected;
	export let onselect;

	function handleClick() {
		onselect(choice, !isSelected);
	}
	function handleKeydown(event) {
		if (event.key === 'Enter') {
			onselect(choice, !isSelected);
		}
	}
</script>

<label>
	<input
		type="checkbox"
		bind:checked={isSelected}
		on:click={handleClick}
		on:keydown={handleKeydown}
	/>
	<span>{choice}</span>
</label>

<style>
	label {
		display: flex;
		margin-bottom: 0.5rem;
	}

	input {
		display: block;
		margin-right: 0.5rem;
	}

	input:checked + span {
		/* background-color: lightblue; */
		background-color: var(--accent);
		color: var(--fg);
		/* background-color: #fc6fbb; */
	}
</style>
