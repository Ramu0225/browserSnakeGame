<template>
	<div>
		<div v-if="!showPlayArea" class="game-container">
			<img
				src="https://see.fontimg.com/api/renderfont4/mLZ3a/eyJyIjoiZnMiLCJoIjo1MSwidyI6MTAwMCwiZnMiOjUxLCJmZ2MiOiIjNzc0QzExIiwiYmdjIjoiI0ZGRkZGRiIsInQiOjF9/V2VsY29tZSB0byBzbmFrZSBnYW1l/terasong.png"
				alt="Welcome to Snake game"
			/>
			<button @click="handleShowPlayerArea" class="btn-welcomepage">
				<img
					src="https://see.fontimg.com/api/renderfont4/mLZ3a/eyJyIjoiZnMiLCJoIjoxNywidyI6MTAwMCwiZnMiOjE3LCJmZ2MiOiIjMDI0MTA2IiwiYmdjIjoiI0ZGRkZGRiIsInQiOjF9/U3RhcnQgR2FtZQ/terasong.png"
					alt="Start Game"
				/>
			</button>
		</div>
		<div v-if="showPlayArea"><PlayArea></PlayArea></div>
	</div>
</template>

<script lang="ts">
import { defineComponent, ref } from "vue";
import PlayArea from "./PlayArea.vue";

export default defineComponent({
	name: "WelcomePage",
	components: {
		PlayArea,
	},
	setup() {
		const showPlayArea = ref(false);

		const handleShowPlayerArea = () => {
			showPlayArea.value = true;
		};
		return {
			showPlayArea,
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
	border: 5px solid rgb(249, 116, 44);
	border-radius: 5px;
	background-color: #33cc00;
	height: 30vh;
	width: 50vw;
}

img {
	width: 80%;
}
</style>
