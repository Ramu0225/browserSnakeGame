<template>
	<div class="container">
		<div class="game">
			<div>
				<canvas width="500" height="500" class="canvas" ref="playArea"></canvas>
			</div>
			<div class="board"></div>
		</div>
	</div>
</template>
<script lang="ts">
import { defineComponent, onMounted, ref, watch, computed } from "vue";
type Direction = "e" | "n" | "s" | "w" | "";
export default defineComponent({
	name: "PlayArea",
	components: {},
	setup(props, { emit }) {
		const playArea = ref<HTMLCanvasElement | null>(null);
		let context: CanvasRenderingContext2D | null;
		const direction = ref<Direction>("");

		const snakeHeadX = ref(120);
		const snakeHeadY = ref(120);
		const score = ref(0);
		const snakeBody = ref([
			[100, 120],
			[110, 120],
			[120, 120],
		]);
		let appleX = 0;
		let appleY = 0;
		let snakeLength = 3;
		let blockSize = 10;
		let speed = 200;
		let interval = 0;

		const handleArrowKeys = (e: KeyboardEvent) => {
			switch (e.key.toLowerCase()) {
				case "arrowright":
				case "d":
					if (direction.value === "w" || direction.value === "e") return;
					pause();
					direction.value = "e";
					break;
				case "arrowdown":
				case "s":
					if (direction.value === "n" || direction.value === "s") return;
					pause();
					direction.value = "s";
					break;
				case "arrowleft":
				case "a":
					if (direction.value === "e" || direction.value === "w") return;
					pause();
					direction.value = "w";
					break;
				case "arrowup":
				case "w":
					if (direction.value === "s" || direction.value === "n") return;
					pause();
					direction.value = "n";
					break;
			}
		};
		const pause = () => {
			clearInterval(interval);
		};
		const drawSnake = () => {
			snakeBody.value.forEach((part) => {
				if (context) {
					context.fillStyle = "black";
					context.fillRect(part[0], part[1], blockSize, blockSize);
				}
			});
		};
		const drawApple = () => {
			if (context) {
				context.fillStyle = "red";
				appleX = getRandomInt();
				appleY = getRandomInt();
				context.fillRect(appleX, appleY, blockSize, blockSize);
			}
		};
		watch(direction, (newDirection) => {
			if (gameOver.value) {
				return;
			}
			//if (newDirection) {
			//this.updateGameStart(true);
			// }
			interval = setInterval(() => excuteDirection(newDirection), speed);
		});
		const excuteDirection = (direction: Direction) => {
			if (gameOver.value) {
				pause();
				return;
			}

			switch (direction) {
				case "e":
					snakeHeadX.value += blockSize;
					break;
				case "s":
					snakeHeadY.value += blockSize;
					break;
				case "w":
					snakeHeadX.value -= blockSize;
					break;
				case "n":
					snakeHeadY.value -= blockSize;
					break;
			}
			moveSnake();
		};
		const moveSnake = () => {
			if (snakeAteApple.value) {
				snakeLength += 1;
				updateScore();
				drawApple();
			}
			if (!context) {
				return;
			}
			if (gameOver.value) {
				return;
			}
			snakeBody.value.push([snakeHeadX.value, snakeHeadY.value]);
			context.fillStyle = "black";
			context.strokeStyle = "yellow";
			context.fillRect(
				snakeHeadX.value,
				snakeHeadY.value,
				blockSize,
				blockSize
			);
			if (snakeBody.value.length > snakeLength) {
				const firstBodyPart = snakeBody.value.shift();
				if (firstBodyPart) {
					context.clearRect(
						firstBodyPart[0],
						firstBodyPart[1],
						blockSize,
						blockSize
					);
				}
			}
		};
		const updateScore = () => {
			score.value++;
		};
		const getRandomInt = () => {
			return Math.floor(Math.random() * 49) * blockSize;
		};
		const snakeAteApple = computed(() => {
			return appleX === snakeHeadX.value && appleY === snakeHeadY.value;
		});
		const gameOver = computed(() => {
			return (
				isSnakeCollide.value ||
				snakeHeadX.value < 0 ||
				snakeHeadY.value < 0 ||
				snakeHeadX.value > 490 ||
				snakeHeadY.value > 490
			);
		});
		const isSnakeCollide = computed(() => {
			return (snakeBody.value || []).some(([x, y], i) => {
				return (
					i !== snakeBody.value.length - 1 &&
					x === snakeHeadX.value &&
					y === snakeHeadY.value
				);
			});
		});
		onMounted(() => {
			window.addEventListener("keydown", handleArrowKeys);
			const canvas = playArea.value;
			if (canvas) {
				context = canvas.getContext("2d");
			}
			drawSnake();
			drawApple();
			//const players = JSON.parse(localStorage.getItem("snakeGame"));
			//this.updateTop10Members(players);
		});
		return { playArea, score };
	},
});
</script>
<style scoped lang="scss">
canvas {
	display: block;
	border: 12px solid rgb(224, 108, 13);
	border-radius: 5px;
	background-color: #33cc00;
	background-image: url(http://www.transparenttextures.com/patterns/45-degree-fabric-dark.png);
}
input {
	padding: 10px;
	outline: none;
	border: 2px solid black;
	border-radius: 5px;
}

.container {
	display: flex;
	flex-direction: column;
	justify-content: center;
	align-items: center;

	.game {
		display: flex;
		flex-direction: row;
		flex-wrap: wrap;
		.board {
			width: 200px;
			background-color: rgb(224, 108, 13);
			margin: 20px 10px 0px;
			border-radius: 5px;
		}
	}
}
</style>
