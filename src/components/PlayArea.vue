<template>
	<PlayerModal
		@close="closeModal"
		:modalActive="modalActive"
		:buttonDisabled="!playerName"
		@restart="restartGame"
		@restartGameOnSwitchingUser="restartGameOnSwitchingUser"
		:gameOverModal="isGameOverDialog"
	>
		<div class="modal-content">
			<div v-if="isGameOverDialog" class="gameover">
				<img
					src="https://see.fontimg.com/api/renderfont4/mLZ3a/eyJyIjoiZnMiLCJoIjozMywidyI6MTAwMCwiZnMiOjMzLCJmZ2MiOiIjMDI0MTA2IiwiYmdjIjoiI0ZGRkZGRiIsInQiOjF9/R2FtZSBPdmVyIQ/terasong.png"
					alt=""
				/>
			</div>
			<div v-else>
				<input
					type="text"
					v-model="playerName"
					maxlength="15"
					placeholder="Enter Player name"
				/>
			</div>
		</div>
	</PlayerModal>
	<div class="container">
		<div class="game">
			<div>
				<PlayerInfo
					:score="score"
					:playerName="playerName"
					:gameOver="gameOver"
					:isPlaying="isPlaying"
					:updateLeaderBoard="updateTop10Members"
					:speed="speed"
				></PlayerInfo>
				<canvas width="500" height="500" class="canvas" ref="playArea"></canvas>
			</div>
			<div class="board">
				<LeaderBoard :players="top10Players"></LeaderBoard>
			</div>
		</div>
	</div>
</template>
<script lang="ts">
import PlayerInfo from "@/components/PlayerInfo.vue";
import LeaderBoard from "@/components/LeaderBoard.vue";
import PlayerModal from "@/components/PlayerModal.vue";
import gsap from "gsap";

import {
	defineComponent,
	onMounted,
	ref,
	watch,
	computed,
	onUnmounted,
} from "vue";

type Direction = "e" | "n" | "s" | "w" | "";
interface PlayerInfoContract {
	name: string;
	score: number;
	timeElapsed: string;
}
export default defineComponent({
	name: "PlayArea",
	components: { PlayerInfo, LeaderBoard, PlayerModal },
	setup() {
		const playArea = ref<HTMLCanvasElement | null>(null);
		let context: CanvasRenderingContext2D | null;
		const direction = ref<Direction>("e");
		let dx = 0;
		let dy = 0;
		const score = ref(0);
		let snakeBody = ref([
			[80, 120],
			[100, 120],
			[120, 120],
		]);
		const playerName = ref("");
		const isPlaying = ref(false);
		const modalActive = ref(true);
		const isGameOverDialog = ref(false);
		let changingDirection = false;
		const startedPlaying = ref(false);
		let appleX = 0;
		let appleY = 0;
		let snakeLength = 3;
		let blockSize = 20;
		let speed = 300;
		let top10Players = ref<Array<PlayerInfoContract>>([]);
		let obstacleCordinates = [0, 0];
		const snakeAteApple = computed(() => {
			const head = snakeBody.value[snakeBody.value.length - 1];
			return appleX === head[0] && appleY === head[1];
		});
		const gameOver = computed(() => {
			const head = snakeBody.value[snakeBody.value.length - 1];
			return (
				isSnakeCollide.value ||
				head[0] < 0 ||
				head[1] < 0 ||
				head[0] > 490 ||
				head[1] > 490 ||
				(head[0] === obstacleCordinates[0] && head[1] === obstacleCordinates[1])
			);
		});
		const isSnakeCollide = computed(() => {
			const head = snakeBody.value[snakeBody.value.length - 1];
			return (snakeBody.value || []).some(([x, y], i) => {
				return (
					i !== snakeBody.value.length - 1 && x === head[0] && y === head[1]
				);
			});
		});
		const updateGameStart = (value: boolean) => {
			isPlaying.value = value;
		};
		const addObstacles = () => {
			if (context) {
				context.clearRect(
					obstacleCordinates[0],
					obstacleCordinates[1],
					blockSize,
					blockSize
				);
				obstacleCordinates = [getRandomInt(), getRandomInt()];
				const hasSnakeCordinates = hasSnakeCoridnates(
					obstacleCordinates[0],
					obstacleCordinates[1]
				);
				if (hasSnakeCordinates) {
					console.log("has snake apple cor");
					addObstacles();
					return;
				}
				var gradient = context.createRadialGradient(
					obstacleCordinates[0] + blockSize / 2,
					obstacleCordinates[1] + blockSize / 2,
					2,
					obstacleCordinates[0] + blockSize / 2,
					obstacleCordinates[1] + blockSize / 2,
					5
				);
				gradient.addColorStop(0, "yellow");
				gradient.addColorStop(1, "red");
				context.beginPath();
				context.arc(
					obstacleCordinates[0] + blockSize / 2,
					obstacleCordinates[1] + blockSize / 2,
					blockSize / 2 - 1,
					0,
					(2 * Math.PI) / 2,
					false
				);
				context.fillStyle = gradient;
				context.fill();
			}
		};
		const handleArrowKeys = (e: KeyboardEvent) => {
			if (changingDirection) return;
			switch (e.key.toLowerCase()) {
				case "arrowright":
				case "d":
					if (direction.value === "w") return;
					excuteDirection("e");
					break;
				case "arrowdown":
				case "s":
					if (direction.value === "n") return;
					excuteDirection("s");
					break;
				case "arrowleft":
				case "a":
					if (direction.value === "e") return;
					excuteDirection("w");
					break;
				case "arrowup":
				case "w":
					if (direction.value === "s") return;
					excuteDirection("n");
					break;
			}
		};

		const drawCircle = (
			x: number,
			y: number,
			radius: number,
			counterClockWise: boolean,
			color: string
		) => {
			if (context) {
				context.fillStyle = color;
				context.beginPath();
				context.arc(x, y, radius, 0, 2 * Math.PI, counterClockWise);
				context.fill();
			}
		};

		const hasSnakeCoridnates = (randomX: number, randomY: number) => {
			return (snakeBody.value || []).some(([x, y]) => {
				return x === randomX && y === randomY;
			});
		};

		const drawSnake = () => {
			snakeBody.value.forEach((part, i) => {
				if (context) {
					const x = part[0] + blockSize / 2;
					const y = part[1] + blockSize / 2;
					const radius = blockSize / 2;
					drawCircle(x, y, radius, false, "black");
					if (snakeBody.value.length - 1 === i) {
						const eye1X = part[0] +blockSize / 1.5;
						const eye1Y = part[1] + blockSize / 3;
						const eyeRadius = 2;
						drawCircle(eye1X, eye1Y, eyeRadius, false, "yellow");
						const eye2X = part[0] +blockSize/ 1.5;
						const eye2Y = part[1] +blockSize/ 1.5;
						drawCircle(eye2X, eye2Y, eyeRadius, false, "yellow");
						context.fillStyle = "red";
						context.lineWidth = 2;
						context.beginPath();
						context.moveTo(part[0] + blockSize / 2, part[1] + blockSize / 2);
						context.lineTo(part[0] + dx, part[1] + dy);
						context.stroke();
						context.fill();
					}
				}
			});
		};
		const drawApple = () => {
			if (context) {
				appleX = getRandomInt();
				appleY = getRandomInt();
				const hasSnakeCordinates = hasSnakeCoridnates(appleX, appleY);
				if (hasSnakeCordinates) {
					console.log("has snake apple cor");
					drawApple();
					return;
				}
				var gradient = context.createRadialGradient(
					appleX + blockSize / 2,
					appleY + blockSize / 2,
					2,
					appleX + blockSize / 2,
					appleY + blockSize / 2,
					5
				);
				gradient.addColorStop(0, "#e88309");
				gradient.addColorStop(1, "#da3e13");
				context.beginPath();
				context.arc(
					appleX + blockSize / 2,
					appleY + blockSize / 2,
					blockSize / 2 - 1,
					0,
					2 * Math.PI,
					false
				);
				context.fillStyle = gradient;
				context.fill();
				context.beginPath();
				context.arc(
					appleX + blockSize / 5,
					appleY + blockSize / 2.5,
					blockSize / 2.5,
					-(2 * Math.PI) / 4,
					0,
					false
				);
				context.fillStyle = "#2f2823";
				context.fill();
			}
		};
		const closeModal = () => {
			modalActive.value = false;
			addEventListners();
		};
		const openModal = () => {
			modalActive.value = true;
			removeEventListners();
		};
		const excuteDirection = (newDirection: Direction) => {
			if (gameOver.value) {
				return;
			}
			changingDirection = true;
			direction.value = newDirection;
			switch (newDirection) {
				case "e":
					dx = blockSize;
					dy = 0;
					break;
				case "s":
					dx = 0;
					dy = blockSize;
					break;
				case "w":
					dx = -blockSize;
					dy = 0;
					break;
				case "n":
					dx = 0;
					dy = -blockSize;
					break;
			}
			updateStartPlay(true);
		};
		const updateTop10Members = (players: Array<PlayerInfoContract>) => {
			if (!players) {
				top10Players.value = [];
				return;
			}
			const sortedMembers = players.sort((a, b) => {
				if (a.score === b.score) {
					const timeElapsed1 = a.timeElapsed
						.split(":")
						.map((t) => Number(t) || 0);
					const timeElapsed2 = b.timeElapsed
						.split(":")
						.map((t) => Number(t) || 0);
					const timeElapsed1Sec = timeElapsed1[0] * 60 + timeElapsed1[1];
					const timeElapsed2Sec = timeElapsed2[0] * 60 + timeElapsed2[1];
					return timeElapsed1Sec - timeElapsed2Sec;
				} else {
					return b.score - a.score;
				}
			});
			if (players.length < 10) {
				top10Players.value = sortedMembers;
			} else {
				top10Players.value = sortedMembers.slice(0, 10);
			}
		};
		const updateStartPlay = (pressedArrowKey: boolean) => {
			startedPlaying.value = pressedArrowKey;
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
			snakeBody.value.push([
				snakeBody.value[snakeBody.value.length - 1][0] + dx,
				snakeBody.value[snakeBody.value.length - 1][1] + dy,
			]);
			if (snakeBody.value.length > snakeLength) {
				const tail = snakeBody.value.shift();
				if (tail) {
					context.clearRect(tail[0], tail[1], blockSize, blockSize);
				}
			}
			drawSnake();
		};
		const updateScore = () => {
			score.value++;
		};
		const updateSpeedLevel = () => {
			speed /= 2;
		};
		const getRandomInt = () => {
			return (
				Math.floor(Math.random() * ((500 - blockSize) / blockSize)) * blockSize
			);
		};
		const clearPlayArea = () => {
			if (!context) {
				return;
			}
			snakeBody.value.forEach((snake) => {
				if (context) {
					context.clearRect(snake[0], snake[1], blockSize, blockSize);
				}
			});
			context.clearRect(appleX, appleY, blockSize, blockSize);
			context.clearRect(
				obstacleCordinates[0],
				obstacleCordinates[1],
				blockSize,
				blockSize
			);
		};
		const updateGameOverDialog = (isOpen: boolean) => {
			isGameOverDialog.value = isOpen;
		};
		const restartGame = () => {
			updateStartPlay(false);
			clearPlayArea();
			snakeBody.value = [
				[80, 120],
				[100, 120],
				[120, 120],
			];
			snakeLength = 3;
			direction.value = "e";
			isPlaying.value = false;
			drawSnake();
			drawApple();
			score.value = 0;
			closeModal();
			updateGameOverDialog(false);
			speed = 300;
			obstacleCordinates = [0, 0];
		};
		const restartGameOnSwitchingUser = () => {
			playerName.value = "";
			restartGame();
			updateGameOverDialog(false);
			openModal();
		};
		const addEventListners = () => {
			if (playerName.value) {
				updateGameStart(true);
				window.addEventListener("keydown", handleArrowKeys);
			}
		};
		const removeEventListners = () => {
			window.removeEventListener("keydown", handleArrowKeys);
		};
		onMounted(() => {
			const canvas = playArea.value;
			if (canvas) {
				context = canvas.getContext("2d");
			}
			drawSnake();
			drawApple();
			const players = localStorage.getItem("snakeGame");
			const parsedPlayers = players ? JSON.parse(players) : [];
			updateTop10Members(parsedPlayers);
		});
		onUnmounted(() => {
			window.removeEventListener("keydown", handleArrowKeys);
		});
		const main = () => {
			if (gameOver.value) {
				return;
			}
			changingDirection = false;
			setTimeout(() => {
				moveSnake();
				main();
			}, speed);
		};

		watch(startedPlaying, () => {
			if (startedPlaying.value) {
				main();
			}
		});
		watch(score, (newScore) => {
			if (newScore !== 0 && newScore % 2 === 0) {
				updateSpeedLevel();
				addObstacles();
			}
		});
		watch(gameOver, (isGameOver) => {
			if (isGameOver) {
				updateGameStart(false);

				gsap.to(".canvas", {
					duration: 0.1,
					x: "+=20",
					yoyo: true,
					repeat: 11,
					ease: "Power2.easeInOut",
					onComplete: () => {
						updateGameOverDialog(true);
						openModal();
					},
				});
			}
		});

		return {
			playArea,
			score,
			playerName,
			gameOver,
			isPlaying,
			updateGameStart,
			top10Players,
			updateTop10Members,
			modalActive,
			closeModal,
			restartGame,
			restartGameOnSwitchingUser,
			isGameOverDialog,
			speed,
		};
	},
});
</script>
<style scoped lang="scss">
canvas {
	width: 500px;
	height: 500px;
	display: block;
	border: 12px solid rgb(224, 108, 13);
	border-radius: 5px;
	background-color: #33cc00;
	background-image: url(http://www.transparenttextures.com/patterns/45-degree-fabric-dark.png);
}
input {
	padding: 6px;
	outline: none;
	border: 2px solid rgb(224, 108, 13);
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
@media (max-width: 500px) {
	canvas {
		width: 250px;
		height: 250px;
	}
	.board {
		display: none;
	}
}
@media (max-width: 740px) {
	canvas {
		width: 300px;
		height: 300px;
	}
}
</style>
