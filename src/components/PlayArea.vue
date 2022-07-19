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

		let snakeHeadX = 120;
		let snakeHeadY = 120;
		let appleX = 0;
		let appleY = 0;
		let snakeLength = 3;
		let blockSize = 10;
		let speed = 200;
		let interval = 0;
		let snakeBody = [
			[100, 120],
			[110, 120],
			[120, 120],
		];

		const handleArrowKeys = (e: KeyboardEvent) => {
			switch (e.key) {
				case "ArrowRight":
					if (direction.value === "w" || direction.value === "e") return;
					direction.value = "e";
					pause();
					break;
				case "ArrowDown":
					if (direction.value === "n" || direction.value === "s") return;
					pause();
					direction.value = "s";
					break;
				case "ArrowLeft":
					if (direction.value === "e" || direction.value === "w") return;
					pause();
					direction.value = "w";
					break;
				case "ArrowUp":
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
			snakeBody.forEach((part) => {
				if (context) {
					context.fillStyle = "black";
					context.fillRect(part[0], part[1], blockSize, blockSize);
				}
			});
		};
		watch(direction, (newDirection) => {
			// if (this.gameOver) {
			// 	return;
			// }
			// if (newDirection) {
			// 	this.updateGameStart(true);
			// }
			interval = setInterval(() => excuteDirection(newDirection), speed);
		});
		const excuteDirection = (direction: Direction) => {
			// if (this.gameOver) {
			// 	this.pause();
			// 	return;
			// }

			switch (direction) {
				case "e":
					snakeHeadX += blockSize;
					break;
				case "s":
					snakeHeadY += blockSize;
					break;
				case "w":
					snakeHeadX -= blockSize;
					break;
				case "n":
					snakeHeadY -= blockSize;
					break;
			}
			moveSnake();
		};
		const moveSnake = () => {
			if (snakeAteApple.value) {
				snakeLength += 1;
				//updateScore();
				//drawApple();
			}
			if (!context) {
				return;
			}
			if (gameOver.value) {
				return;
			}
			snakeBody.push([snakeHeadX, snakeHeadY]);

			context.fillStyle = "black";
			context.strokeStyle = "yellow";
			context.fillRect(snakeHeadX, snakeHeadY, blockSize, blockSize);
			if (snakeBody.length > snakeLength) {
				const firstBodyPart = snakeBody.shift();
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
		const snakeAteApple = computed(() => {
			return appleX === snakeHeadX && appleY === snakeHeadY;
		});
		const gameOver = computed(() => {
			return (
				isSnakeCollide.value ||
				snakeHeadX < 0 ||
				snakeHeadY < 0 ||
				snakeHeadX > 490 ||
				snakeHeadY > 490
			);
		});
		const isSnakeCollide = computed(() => {
			return (snakeBody || []).some(([x, y], i) => {
				return (
					i !== snakeBody.length - 1 && x === snakeHeadX && y === snakeHeadY
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
		});
		return { playArea };
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
