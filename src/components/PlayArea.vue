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
			<div v-if="isGameOverDialog">
				<div>
					<h1>Game over</h1>
				</div>
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
				></PlayerInfo>
				<canvas width="500" height="500" class="canvas" ref="playArea"></canvas>
			</div>
			<div class="board">
				<LeaderBoard :list="top10Players"></LeaderBoard>
			</div>
		</div>
	</div>
</template>
<script lang="ts">
import PlayerInfo from "@/components/PlayerInfo.vue";
import LeaderBoard from "@/components/LeaderBoard.vue";
import PlayerModal from "@/components/PlayerModal.vue";

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
		const direction = ref<Direction>("");

		const snakeHeadX = ref(120);
		const snakeHeadY = ref(120);
		const score = ref(0);
		const snakeBody = ref([
			[100, 120],
			[110, 120],
			[120, 120],
		]);
		const playerName = ref("");
		const isPlaying = ref(false);
		const modalActive = ref(false);
		const isGameOverDialog = ref(false);
		let appleX = 0;
		let appleY = 0;
		let snakeLength = 3;
		let blockSize = 10;
		let speed = 200;
		let interval = 0;
		let top10Players: Array<PlayerInfoContract> = [];
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
		const updateGameStart = (value: boolean) => {
			isPlaying.value = value;
		};
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
		const closeModal = () => {
			modalActive.value = false;
			addEventListners();
		};
		const openModal = () => {
			modalActive.value = true;
		};
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
		const updateTop10Members = (players: Array<PlayerInfoContract>) => {
			if (!players) {
				top10Players = [];
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
				top10Players = sortedMembers;
			} else {
				top10Players = sortedMembers.slice(0, 10);
			}
		};
		watch(gameOver, (isGameOver) => {
			if (isGameOver) {
				updateGameStart(false);
				updateGameOverDialog(true);
				openModal();
			}
		});
		watch(direction, (newDirection) => {
			if (gameOver.value) {
				return;
			}
			interval = setInterval(() => excuteDirection(newDirection), speed);
		});
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
		};
		const updateGameOverDialog = (isOpen: boolean) => {
			isGameOverDialog.value = isOpen;
		};

		const restartGame = () => {
			clearPlayArea();
			snakeBody.value = [
				[100, 120],
				[110, 120],
				[120, 120],
			];
			snakeLength = 3;
			direction.value = "";
			isPlaying.value = false;
			snakeHeadX.value = 120;
			snakeHeadY.value = 120;
			drawSnake();
			drawApple();
			score.value = 0;
			updateGameOverDialog(false);
			closeModal();
		};
		const restartGameOnSwitchingUser = () => {
			restartGame();
			playerName.value = "";
			updateGameOverDialog(false);
			openModal();
		};
		const addEventListners = () => {
			if (playerName.value) {
				updateGameStart(true);
				window.addEventListener("keydown", handleArrowKeys);
			}
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
			console.log(parsedPlayers);
			updateTop10Members(parsedPlayers);
			openModal();
		});
		onUnmounted(() => {
			window.removeEventListener("keydown", handleArrowKeys);
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
		};
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
