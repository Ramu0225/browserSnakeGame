<template>
	<div>
		<div v-if="!showPlayArea" class="game-container">
			<h1>Welcome to Snake Game</h1>
			<button @click="handleShowPlayerArea">Start Game</button>
		</div>
	</div>
</template>

<script lang="ts">
import { defineComponent, ref } from "vue";
export default defineComponent({
	name: "WelcomePage",
	props: {
		msg: String,
	},
	components: {
		
	},
	setup() {
		const showPlayerNamePrompt = ref(false);
		const showPlayArea = ref(false);
		const playerName = ref("");
		const handleShowPlayerPromptClick = () => {
			showPlayerNamePrompt.value = true;
		};
		const handleShowPlayerArea = () => {
			showPlayerNamePrompt.value = false;
			showPlayArea.value = true;
		};
		return {
			showPlayerNamePrompt,
			showPlayArea,
			playerName,
			handleShowPlayerPromptClick,
			handleShowPlayerArea,
		};
	},
});
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
h3 {
	margin: 40px 0 0;
}
.game-container {
	display: flex;
	flex-direction: column;
	justify-content: space-around;
	align-items: center;
	margin: 20px auto;
	border: 5px solid rgb(224, 108, 13);
	border-radius: 5px;
	background-color: #33cc00;
	height: 30vh;
	width: 50vw;
}
button {
	padding: 10px 20px;
	border: none;
	background-color: rgb(100, 6, 72);
	border-radius: 5px;
	color: #fff;
	cursor: pointer;
	width: 50%;
}
</style>
