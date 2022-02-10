<script lang="ts">
	enum GameState {
		Initial = 0,
		Playing = 1,
		Paused = 2,
	}

	enum Cell {
		Empty = 0,
		Snake = 1,
		Food = 2,
	}

	enum Direction {
		Up = 1,
		Down = 2,
		Left = 3,
		Right = 4,
	}

	interface Coordinate {
		x: number;
		y: number;
	}

	const boardSize = 15;
	const rows = Array<Array<Cell>>(boardSize);
	let gameState = GameState.Initial;
	let snakeCells: Coordinate[] = [];
	let score = 0;
	let timer: NodeJS.Timer;
	let currDirection = Direction.Right;

	function resetGame() {
		for (let i = 0; i < boardSize; i++) {
			const cells = Array<Cell>(boardSize).fill(Cell.Empty);
			rows[i] = cells;
		}
		const mid = Math.floor(boardSize / 2);
		rows[mid][mid] = Cell.Snake;
		snakeCells.length = 0;
		snakeCells.push({ x: mid, y: mid });
		currDirection = Direction.Right;
		score = 0;
		gameState = GameState.Initial;
	}

	const directionByKey = {
		ArrowRight: Direction.Right,
		ArrowLeft: Direction.Left,
		ArrowUp: Direction.Up,
		ArrowDown: Direction.Down,
		d: Direction.Right,
		a: Direction.Left,
		w: Direction.Up,
		s: Direction.Down,
	};

	function handleKey(event: KeyboardEvent) {
		const nextDirection: Direction = directionByKey[event.key];
		if (nextDirection === undefined) {
			return;
		}
		if (
			(currDirection == Direction.Left && nextDirection == Direction.Right) ||
			(currDirection == Direction.Right && nextDirection == Direction.Left) ||
			(currDirection == Direction.Up && nextDirection == Direction.Down) ||
			(currDirection == Direction.Down && nextDirection == Direction.Up)
		) {
			return;
		}
		currDirection = nextDirection;
	}

	function startGame() {
		window.addEventListener('keydown', handleKey);
		spawnFood();
		continueGame();
	}

	function pauseGame() {
		window.removeEventListener('keydown', handleKey);
		gameState = GameState.Paused;
		clearInterval(timer);
	}

	function continueGame() {
		gameState = GameState.Playing;
		timer = setInterval(moveSnake, 200);
	}

	function moveSnake() {
		const curr = snakeCells[snakeCells.length - 1];
		const next = nextCoordinate(curr, currDirection);
		if (!isSafeCoordinate(next)) {
			pauseGame();
			resetGame();
			alert('You lose!');
			return;
		}
		if (rows[next.y][next.x] === Cell.Food) {
			score++;
			spawnFood();
		} else {
			const tail = snakeCells.shift();
			rows[tail.y][tail.x] = Cell.Empty;
		}
		snakeCells.push(next);
		rows[next.y][next.x] = Cell.Snake;
	}

	function nextCoordinate(coordinate: Coordinate, direction: Direction): Coordinate {
		const result = {
			x: coordinate.x,
			y: coordinate.y,
		};
		switch (direction) {
			case Direction.Left:
				result.x -= 1;
				break;
			case Direction.Right:
				result.x += 1;
				break;
			case Direction.Up:
				result.y -= 1;
				break;
			case Direction.Down:
				result.y += 1;
				break;
		}
		return result;
	}

	function isSafeCoordinate(coordinate: Coordinate): boolean {
		return !(
			coordinate.x < 0 ||
			coordinate.x === boardSize ||
			coordinate.y < 0 ||
			coordinate.y === boardSize ||
			rows[coordinate.y][coordinate.x] === Cell.Snake
		);
	}

	function getRandomNumber(min: number, max: number): number {
		return Math.floor(Math.random() * (max - min + 1)) + min;
	}

	function spawnFood() {
		let x = getRandomNumber(0, boardSize);
		let y = getRandomNumber(0, boardSize);
		if (rows[y][x] !== Cell.Empty) {
			x = getRandomNumber(0, boardSize);
			y = getRandomNumber(0, boardSize);
		}
		rows[y][x] = Cell.Food;
	}

	resetGame();
</script>

<main>
	<h1>Snake</h1>
	<div id="board">
		{#each rows as row}
			{#each row as cell}
				<div class="cell" class:snake={cell === Cell.Snake} class:food={cell === Cell.Food} />
			{/each}
		{/each}
	</div>
	<hr />
	<button on:click={startGame} class:hidden={gameState !== GameState.Initial}>Start</button>
	<button on:click={pauseGame} class:hidden={gameState !== GameState.Playing}>Pause</button>
	<button on:click={continueGame} class:hidden={gameState !== GameState.Paused}>Continue</button>
	<button on:click={resetGame} class:hidden={gameState !== GameState.Paused}>Reset</button>
	<p>Score: {score}</p>
</main>

<style>
	main {
		display: flex;
		flex-direction: column;
		align-items: center;
	}

	#board {
		display: grid;
		height: 60vmin;
		width: 60vmin;
		border: 1px solid #ccc;
		grid-template-columns: repeat(15, 1fr);
		grid-template-rows: repeat(15, 1fr);
	}

	.cell {
		border: 0.05px solid #ccc;
	}

	.snake {
		background-color: green;
	}

	.food {
		background-color: red;
	}

	.hidden {
		display: none;
	}
</style>
