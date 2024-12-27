<script>
	import Answer from './Answer.svelte';
	export let question;
	export let choices;
	export let correctAnswer;
	export let index;
	export let selectedAnswer;
	export let onselectAnswer;

	function handleSelect(answer) {
		onselectAnswer(new CustomEvent('selectAnswer', { detail: answer }));
	}

	$: randomizedChoices = [...choices].sort(() => Math.random() - 0.5);
</script>

<div class="question" id={index}>
	<p class="qNum">{index + 1}</p>
	<p class="gradient-border">{question}</p>
	{#each randomizedChoices as choice, i}
		<Answer {choice} isSelected={selectedAnswer === choice} onselect={handleSelect} />
	{/each}
</div>
<div class="correct-answer hidden">X</div>
<div class="wrong-answer hidden">X</div>

<style>
	.question {
		/* font-family:
			system-ui,
			-apple-system,
			BlinkMacSystemFont,
			'Segoe UI',
			Roboto,
			Oxygen,
			Ubuntu,
			Cantarell,
			'Open Sans',
			'Helvetica Neue',
			sans-serif; */
		margin-bottom: 3rem;
		width: fit-content;
		border: 1px solid black;
		border-radius: 7px;
		padding: 10px;
		box-shadow: 10px 10px 5px 0px rgba(0, 0, 0, 0.75);
		background: #ffffff80;
	}
	.qNum {
		transform: rotate(-15deg);
		display: flex;
		justify-content: center;
		align-items: center;
		font-size: 1.5rem;
		font-weight: bold;
		border: 3px solid limegreen;
		padding: 15px;
		width: 20px;
		height: 20px;
		border-radius: 50%;
		z-index: 3;
		margin-top: -30px;
		margin-left: -30px;
		background-color: limegreen;
		/* box-shadow:
			25px 25px 75px rgba(0, 0, 0, 0.25),
			10px 10px 70px rgba(0, 0, 0, 0.25),
			inset 5px 5px 10px rgba(0, 0, 0, 0.5),
			inset 5px 5px 20px rgba(255, 255, 255, 0.2),
			inset -5px -5px 15px rgba(0, 0, 0, 0.75); */

		box-shadow:
			5px 5px 7px rgba(0, 0, 0, 0.25),
			inset -3px -3px 5px rgba(0, 0, 0, 0.5);
	}
	.question:hover {
		background: #ffffff90;
		box-shadow: 10px 10px 5px 0px rgba(0, 0, 0, 0.95);
	}

	p {
		margin-bottom: 0.5rem;
		font-weight: bold;
		text-transform: uppercase;
		margin-bottom: 10px;
		border-bottom: 1px solid black;
	}
	.gradient-border {
		border-bottom: 1px solid;
		/* border-image: linear-gradient(to right, red, orange, yellow, green, blue, purple); */
		border-image: linear-gradient(to right, #58c9e8, #005677) 1;
		border-image-slice: 1;
		padding-bottom: 2px;
		/* max-width: 600px; */
	}
	.correct-answer {
		background-color: lightgreen;
		color: var(--black);
	}
	.correct-answer:hover {
		background-color: lightgreen;
	}
	.wrong-answer {
		background-color: #ff7f7f;
	}
	.wrong-answer:hover {
		background-color: #ff7f7f;
	}
	.hidden {
		display: none;
	}
</style>
