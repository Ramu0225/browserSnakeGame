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
				<h1>Game over!</h1>
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
				<LeaderBoard :players="top10Players"></LeaderBoard>
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
		const direction = ref<Direction>("e");

		const snakeHeadX = ref(120);
		const snakeHeadY = ref(120);
		let dx = 0;
		let dy = 0;
		const score = ref(0);
		let snakeBody = ref([
			[100, 120],
			[110, 120],
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
		let blockSize = 10;
		let speed = 200;
		let top10Players = ref<Array<PlayerInfoContract>>([]);
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
				head[1] > 490
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
				appleX = getRandomInt();
				appleY = getRandomInt();
				context.fillStyle = "red";
				// context.fillRect(appleX, appleY, blockSize, blockSize);

				context.beginPath();
				context.arc(appleX + 5, appleY + 5, 4, 0, 2 * Math.PI, false);
				context.fill();
				// var img = new Image();
				// img.src = "./assets/logo.png"; // Put the path to you SVG image here.
				// img.onload = function () {
				// 	if (context) {
				// 		context.drawImage(img, appleX, appleY);
				// 	}
				// };
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
					dx = 10;
					dy = 0;
					break;
				case "s":
					dx = 0;
					dy = 10;
					break;
				case "w":
					dx = -10;
					dy = 0;
					break;
				case "n":
					dx = 0;
					dy = -10;
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
		watch(gameOver, (isGameOver) => {
			if (isGameOver) {
				updateGameStart(false);
				updateGameOverDialog(true);
				openModal();
			}
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
			snakeBody.value.push([
				snakeBody.value[snakeBody.value.length - 1][0] + dx,
				snakeBody.value[snakeBody.value.length - 1][1] + dy,
			]);
			drawSnake();
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
			updateStartPlay(false);
			clearPlayArea();
			snakeHeadX.value = 120;
			snakeHeadY.value = 120;
			snakeBody.value = [
				[100, 120],
				[110, 120],
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
	padding: 6px;
	outline: none;
	border: 2px solid rgb(224, 108, 13);
	border-radius: 5px;
}
.gameover {
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
