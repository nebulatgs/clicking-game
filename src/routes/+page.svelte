<script lang="ts">
	const ROUNDS = 4;
	const MIN_WAIT_MS = 1_000;
	const MAX_WAIT_MS = 5_000;

	enum State {
		Start,
		Click,
		Present,
		Wait,
		TooFast,
		GameEnd
	}

	let state: State = State.Start;
	let startTime = Date.now();
	let durations: number[] = [];
	let currentRound = 0;
	let timeoutId: NodeJS.Timeout;

	const colors: Record<State, string> = {
		[State.Start]: 'bg-[#2B87D1]',
		[State.Wait]: 'bg-[#CE2636]',
		[State.Click]: 'bg-[#4BDB6A]',
		[State.Present]: 'bg-[#2B87D1]',
		[State.TooFast]: 'bg-[#2B87D1]',
		[State.GameEnd]: 'bg-[#2B87D1]'
	};

	$: texts = {
		[State.Start]: 'Click to start',
		[State.Wait]: 'Wait',
		[State.Click]: 'Click',
		[State.Present]: `${durations[durations.length - 1]}ms`,
		[State.TooFast]: 'Too fast!',
		[State.GameEnd]: `${Math.round(durations.reduce((p, c) => p + c, 0) / ROUNDS)}ms on avg.`
	};

	function play() {
		if (currentRound < ROUNDS) {
			currentRound += state === State.TooFast ? 0 : 1;
			state = State.Wait;
			timeoutId = setTimeout(() => {
				state = State.Click;
				startTime = Date.now();
			}, Math.random() * (MAX_WAIT_MS - MIN_WAIT_MS) + MIN_WAIT_MS);
		} else {
			state = State.GameEnd;
			currentRound = 0;
		}
	}

	const actions = {
		[State.Start]() {
			durations = [];
			play();
		},
		[State.Wait]() {
			state = State.TooFast;
		},
		[State.Click]() {
			durations.push(Date.now() - startTime);
			durations = durations;
			state = State.Present;
		},
		[State.Present]() {
			play();
		},
		[State.TooFast]() {
			clearTimeout(timeoutId);
			play();
		},
		[State.GameEnd]() {
			state = State.Start;
		}
	};

	function click() {
		actions[state]();
	}
</script>

<div
	tabIndex={0}
	class="{colors[state]} grid place-items-center w-screen h-screen"
	on:mousedown={click}
	on:keydown={click}
>
	<h1 class="text-6xl font-semibold select-none text-white">
		{texts[state]}
	</h1>
</div>
