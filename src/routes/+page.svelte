<script>
	// export let PageData;
	// console.log(PageData);
	// export let userID = PageData.userID;
	import Question from '../lib/components/Question.svelte';
	import { questions } from '../lib/data/questions.js';

	import { ConfettiCannon } from 'svelte-canvas-confetti';
	import { onMount, tick } from 'svelte';
	const makeConfettiCannon = async () => {
		confettiCannon = false;
		await tick();
		confettiCannon = true;
	};
	let confettiCannon = false;

	let answers = new Array(questions.length).fill(null);
	// console.log(answers);

	let score = 0;
	let total = questions.length;
	let passing = total * 0.8;
	let passed = false;
	let userID = null;
	let toStudy = [];
	let message = '';
	let eligible;
	function checkEligibility() {
		userID = getUserIdFromLocalStorage();
		const data = {
			userID: userID
		};
		fetch('/API/checkMPTestEligibility.php', {
			method: 'POST',
			headers: {
				'Content-Type': 'application/json'
			},
			body: JSON.stringify(data)
		})
			.then((response) => response.json())
			.then((data) => {
				if (data.eligible) {
					console.log('Eligible');
					eligible = true;
				} else {
					console.log('Not eligible');
					eligible = false;
				}
			});
	}
	onMount(() => {
		checkEligibility();
	});

	function recordAttempt() {
		userID = getUserIdFromLocalStorage();
		const data = {
			userID: userID
		};
		fetch('/API/submitMPTestFail.php', {
			method: 'POST',
			headers: {
				'Content-Type': 'application/json'
			},
			body: JSON.stringify(data)
		});
	}
	function submitAnswers() {
		score = 0;

		for (let i = 0; i < answers.length; i++) {
			let answer = answers[i];
			let correctAnswer = questions[i].correctAnswer;
			let questionIndex = questions[i].index;
			let manualSection = questions[i].manualSection;
			let questionDiv = document.getElementById(questionIndex);

			if (answer === correctAnswer) {
				questionDiv.classList.add('correct-answer');
				questionDiv.classList.remove('wrong-answer');
				score++;
			} else {
				toStudy.push({ manualSection: manualSection });
				questionDiv.classList.remove('correct-answer');
				questionDiv.classList.add('wrong-answer');
			}
		}

		if (score >= passing) {
			makeConfettiCannon();
			passed = true;
			//submitScore();
		}
		if (score < passing) {
			// alert(`You got ${score} out of ${answers.length} correct!`);
			recordAttempt();
			var messagePopover = document.getElementById('messagePopover');
			message += 'You did not pass the test. Please try again after studying manual sections:';
			for (let i = 0; i < toStudy.length; i++) {
				message += '\n' + toStudy[i].manualSection + ',';
			}
			messagePopover.setAttribute('data-content', message);
			messagePopover.showPopover();
			return message;
			// alert(message);
		}
	}

	function showAnswers() {
		for (let i = 0; i < questions.length; i++) {
			answers[i] = questions[i].correctAnswer;
		}
	}
	function reset() {
		const wrongAnswersDivs = document.querySelectorAll('div.wrong-answer');
		wrongAnswersDivs.forEach((div) => {
			div.classList.remove('wrong-answer');
		});

		answers = new Array(questions.length).fill(null);
		score = 0;
		const correctAnswerDivs = document.querySelectorAll('div.correct-answer');
		correctAnswerDivs.forEach((div) => {
			div.classList.remove('correct-answer');
		});
		message = '';
	}
	function getUserIdFromLocalStorage() {
		let user = JSON.parse(localStorage.getItem('bcdash-userData'));
		if (user) {
			console.log(user);
			userID = user.id;
			return userID;
		} else {
			console.error('User data not found or invalid');
		}
	}
	// onMount(async () => {
	// 	const user = JSON.parse(localStorage.getItem('bcdash_userData'));
	// 	if (user && user.id) {
	// 		userID = user.id;
	// 	} else {
	// 		console.error('User data not found or invalid');
	// 	}
	// });

	// let userID = getUserIdFromLocalStorage();

	function submitScore() {
		userID = getUserIdFromLocalStorage();
		// let userID = getUserIdFromLocalStorage();
		//alert(`You got ${score} out of ${answers.length} correct!`);
		const data = {
			userID: userID
		};
		fetch('/API/submitMPTestScore.php', {
			method: 'POST',
			headers: {
				'Content-Type': 'application/json'
			},
			body: JSON.stringify(data)
		});
	}

	$: {
		const _ = score;
	}
</script>

{#if !eligible}
	<div class="body">
		<div class="mute">
			<h2>Motor Pool Test</h2>
			<p>You have already taken the test today. Please try again tomorrow.</p>
		</div>
		<div class="mute">
			<a href="/">Return to Dashboard</a>
		</div>
	</div>
{:else}
	<div class="body">
		<div class="mute">
			<h2>Motor Pool Test</h2>
			<div class="questions">
				{#each questions as { question, choices, correctAnswer }, index}
					<Question
						{question}
						{choices}
						{index}
						selectedAnswer={answers[index]}
						onselectAnswer={(event) => (answers[index] = event.detail)}
					/>
					<p class="answer">Answer: {correctAnswer}</p>
				{/each}
			</div>
			<div class="buttons">
				{#if !passed}
					<button on:click={submitAnswers}>Submit</button>
					<button on:click={reset}>Reset</button>
					<!-- <button on:click={showAnswers}>Show Me</button> -->
				{/if}
				{#if passed}
					<p>Your score has been submitted. You may close this window now.</p>
					<!-- <button on:click={reset}>Dev Reset</button> -->
				{/if}

				{#if confettiCannon}
					<ConfettiCannon
						origin={[window.innerWidth / 2, window.innerHeight]}
						angle={-90}
						spread={35}
						force={35}
					/>
				{/if}
			</div>

			<p class="score">Score: {score} of {total}</p>
		</div>
	</div>
	<div class="messagePopover" name="messagePopover" id="messagePopover" popover="manual">
		{message}
		<div class="arrow">
			<button on:click={reset} popovertarget="messagePopover" popovertargetaction="hide"
				>Close</button
			>
		</div>
	</div>
{/if}

<style>
	.body {
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
		margin: 0;
		padding: 20px;
		/* background-image: linear-gradient(90deg, #00dbde 0%, #fc00ff 100%); */
		/* background: radial-gradient(circle, violet, indigo, blue, green, yellow, orange, red); */
		background-position: center center;
		background-size: cover;
		background-attachment: fixed;
		/* height: 100dvh; */
	}
	.mute {
		/* background: #ffffff70; */
		background-color: var(--bg);
		border-radius: 7px;
		padding: 20px;
		box-shadow: 10px 10px 5px 0px rgba(0, 0, 0, 0.75);
	}
	.questions {
		display: flex;
		flex-wrap: wrap;
		flex-basis: auto;
		align-content: space-around;
		margin: 10px;
		padding: 20px;
		/* display: flex; */
		gap: 2rem;
		align-items: center;
	}
	.answer {
		display: none;
	}
	h2 {
		text-align: center;
		text-transform: uppercase;
		font-weight: bolder;
		font-size: xx-large;
		color: var(--fg);
		/* color: #ffffff; */
		/* margin-bottom: 20px; */
	}
	/* h3 {
		text-align: center;
		text-transform: uppercase;
		font-weight: bolder;
		font-size: large;
		color: #ffffff;
		margin-bottom: 20px;
	} */
	.score {
		text-align: center;
		font-weight: bolder;
		font-size: xx-large;
		color: #ffffff;
	}
	.buttons {
		display: flex;
		justify-content: center;
	}
	button {
		cursor: pointer;
		border: none;
		border-radius: 4px;
		padding: 0.75rem 2rem;
		margin: 1rem;
		font-weight: bold;
		font-size: 1rem;
		font-family: 'Gill Sans', 'Gill Sans MT', Calibri, 'Trebuchet MS', sans-serif;
		color: var(--fg);
		background: linear-gradient(-45deg, hsl(193, 76%, 63%), hsl(197, 100%, 23%));
		box-shadow: 5px 5px 0 0 rgba(0, 0, 0, 0.8);
		text-shadow: -1px -1px rgba(0, 0, 0, 0.6);
		transition:
			transform 0.1s ease-in-out,
			box-shadow 0.1s ease-in-out;
	}

	button:hover {
		/* background: linear-gradient(-45deg, hsl(193, 76%, 63%), hsl(300, 95%, 65%)); */
		background: linear-gradient(45deg, hsl(193, 76%, 63%), hsl(197, 100%, 23%));
	}

	button:active {
		transform: translate(2px, 2px);
		box-shadow: 3px 3px 0 0 rgba(0, 0, 0, 0.8);
	}
	/* popover {
		inset: auto;
	} */
	.messagePopover {
		inset: auto;
		color: var(--fg);
		background: var(--bg);
		width: fit-content;
		position: fixed;
		left: 50%;
		top: 50%;
		transform: translate(-50%, -50%);
		padding: 20px;
		/* backdrop-filter: blur(10px); */
	}
	::backdrop {
		backdrop-filter: blur(5px) !important;
	}
</style>
