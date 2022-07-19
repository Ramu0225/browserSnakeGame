<template>
	<div class="info-container">
		<div class="info">
			<div class="square">
				<div class="icon-circle">C</div>
				<span>{{ playerName || "Player" }}</span>
			</div>
			<div class="square">
				<div class="icon-circle">C</div>
				{{ score }}
			</div>
		</div>
		<div class="square">
			<div class="icon-circle">C</div>
			{{ `${displayGameTimeMinutes}:${displayGameTimeSeconds}` }}
		</div>
	</div>
</template>

<script>
import { ref, watch } from "vue";
export default {
	props: [
		"playerName",
		"score",
		"gameOver",
		"isPlaying",
		"updateLeaderBoard",
		"updateGameStart",
	],
	setup(props, { emit }) {
		let interval = 0;
		let gameStartTime = 0;
		let gameEndTime = 0;
		let displayGameTimeMinutes = ref("00");
		let displayGameTimeSeconds = ref("00");
		//const { name, score } = props;
		watch(
			() => props.gameOver,
			(isGameOver) => {
				if (isGameOver) {
					stopTimer();
					// TODO: get top10 members from backend and update the members every 1min
					const snakeGameMembers = JSON.parse(
						localStorage.getItem("snakeGame")
					);
					if (snakeGameMembers) {
						snakeGameMembers.push({
							name: props.playerName,
							score: props.score,
							timeElapsed: `${displayGameTimeMinutes.value}:${displayGameTimeSeconds.value}`,
						});
						localStorage.setItem("snakeGame", JSON.stringify(snakeGameMembers));
						props.updateLeaderBoard(snakeGameMembers);
					}
					props.updateGameStart(false);
				}
			}
		);
		watch(
			() => props.isPlaying,
			(isPlaying) => {
				if (isPlaying) {
					setGameStartTimer();
					startTimer();
				} else {
					updateDisplaySeconds(0);
					updateDisplayMinutes(0);
				}
			}
		);
		const updateDisplayTimes = () => {
			const gameTimeDiff = timeDiff();
			const gameTimeMilliSeconds = gameTimeDiff / 1000;
			const gameTimeSeconds = Math.floor(gameTimeMilliSeconds % 60);
			const gameTimeMinutes = Math.floor(gameTimeMilliSeconds / 60);
			updateDisplayMinutes(gameTimeMinutes);
			updateDisplaySeconds(gameTimeSeconds);
		};
		const updateDisplaySeconds = (seconds) => {
			displayGameTimeSeconds.value =
				seconds.toString().length === 1 ? `0${seconds}` : seconds;
		};
		const updateDisplayMinutes = (minutes) => {
			displayGameTimeMinutes.value =
				minutes.toString().length === 1 ? `0${minutes}` : minutes;
		};
		const setEndTimer = () => {
			gameEndTime = new Date();
			updateDisplayTimes();
		};
		const setGameStartTimer = () => {
			gameStartTime = new Date();
		};
		const startTimer = () => {
			interval = setInterval(setEndTimer, 1000);
		};
		const stopTimer = () => {
			clearInterval(interval);
		};

		const timeDiff = () => {
			return gameEndTime.getTime() - gameStartTime.getTime();
		};

		return {
			startTimer,
			stopTimer,
			gameStartTime,
			gameEndTime,
			timeDiff,
			displayGameTimeMinutes,
			displayGameTimeSeconds,
			updateDisplaySeconds,
			updateDisplayMinutes,
		};
	},
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
.info-container {
	display: flex;
	flex-direction: row;
	justify-content: space-between;
	margin-bottom: 2px;

	.info {
		width: 200px;
		display: flex;
		justify-content: space-between;
	}
	.square {
		width: 60px;
		background-color: gold;
		padding: 5px 10px;
		border-radius: 15px;
		font-size: 10px;
		position: relative;
	}
	.icon-circle {
		position: absolute;
		top: 0;
		left: -5px;
		z-index: 2;
		display: flex;
		flex-direction: row;
		justify-content: center;
		align-items: center;
		width: 20px;
		height: 20px;
		background-color: red;
		border: 1px solid black;
		border-radius: 50%;
	}
}
</style>
