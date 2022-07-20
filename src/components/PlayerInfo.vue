<template>
	<div class="info-container">
		<div class="info">
			<div class="square">
				<div class="icon-circle">
					<img
						src="https://img.icons8.com/pastel-glyph/20/000000/person-male--v3.png"
					/>
				</div>
				<span>{{ playerName || "Player" }}</span>
			</div>
			<div class="square">
				<div class="icon-circle">
					<img src="https://img.icons8.com/doodle/20/000000/apple.png" />
				</div>
				{{ score }}
			</div>
		</div>
		<div class="square">
			<div class="icon-circle">
				<img src="https://img.icons8.com/ios/18/000000/hourglass.png" />
			</div>
			{{ `${displayGameTimeMinutes}:${displayGameTimeSeconds}` }}
		</div>
	</div>
</template>

<script lang="ts">
import { ref, watch, defineComponent } from "vue";
export default defineComponent({
	props: {
		playerName: String,
		score: Number,
		gameOver: Boolean,
		isPlaying: Boolean,
		updateLeaderBoard: Function,
	},
	setup(props) {
		let interval = 0;
		let gameStartTime = new Date();
		let gameEndTime = new Date();
		let displayGameTimeMinutes = ref("00");
		let displayGameTimeSeconds = ref("00");
		watch(
			() => props.isPlaying,
			(isPlaying) => {
				if (isPlaying) {
					setGameStartTimer();
					startTimer();
				} else {
					stopTimer();
					// TODO: get top10 members from backend and update the members every 1min
					const players = localStorage.getItem("snakeGame");
					const snakeGameMembers = players ? JSON.parse(players) : [];
					if (snakeGameMembers) {
						snakeGameMembers.push({
							name: props.playerName,
							score: props.score,
							timeElapsed: `${displayGameTimeMinutes.value}:${displayGameTimeSeconds.value}`,
						});
						localStorage.setItem("snakeGame", JSON.stringify(snakeGameMembers));
						if (props.updateLeaderBoard) {
							props.updateLeaderBoard(snakeGameMembers);
						}
					}
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
		const updateDisplaySeconds = (seconds: number) => {
			displayGameTimeSeconds.value =
				seconds.toString().length === 1 ? `0${seconds}` : `${seconds}`;
		};
		const updateDisplayMinutes = (minutes: number) => {
			displayGameTimeMinutes.value =
				minutes.toString().length === 1 ? `0${minutes}` : `${minutes}`;
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
});
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
		background-color: #33cc00;
		margin: 1px;
		padding: 4.5px 0px 4.5px 10px;
		border-radius: 15px;
		font-size: 10px;
		font-weight: bold;
		position: relative;
		font-family: "Lucida Sans", "Lucida Sans Regular", "Lucida Grande",
			"Lucida Sans Unicode", Geneva, Verdana, sans-serif;
	}
	.icon-circle {
		position: absolute;
		top: 0;
		//bottom:2px;
		left: 0px;
		//padding: 0px;
		//margin-bottom: 12px;
		z-index: 2;
		display: flex;
		flex-direction: row;
		justify-content: center;
		align-items: center;
		width: 20px;
		height: 20px;
		//background-color: rgb(224, 108, 13);
		border: 1px solid #33cc00;
		border-radius: 50%;
	}
}
</style>
