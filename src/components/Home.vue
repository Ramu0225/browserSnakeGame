<template>
	<div>
		<div v-if="!showPlayArea" class="game-container">
			<img
				src="https://see.fontimg.com/api/renderfont4/mLZ3a/eyJyIjoiZnMiLCJoIjo1MSwidyI6MTAwMCwiZnMiOjUxLCJmZ2MiOiIjNzc0QzExIiwiYmdjIjoiI0ZGRkZGRiIsInQiOjF9/V2VsY29tZSB0byBzbmFrZSBnYW1l/terasong.png"
				alt="Welcome to Snake game"
			/>
			<button @click="handleShowPlayerArea">
				<img
					src="https://see.fontimg.com/api/renderfont4/mLZ3a/eyJyIjoiZnMiLCJoIjoxNywidyI6MTAwMCwiZnMiOjE3LCJmZ2MiOiIjMDI0MTA2IiwiYmdjIjoiI0ZGRkZGRiIsInQiOjF9/U3RhcnQgR2FtZQ/terasong.png"
					alt=""
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

button {
	padding: 10px 20px;
	border: 1px solid #ac6c41;
	box-shadow: inset 0px 0px 0px 5px #97b738;
	border-radius: 30px;
	background-image: linear-gradient(
		rgb(231, 215, 215) 0,
		rgba(209, 88, 24, 0.4) 100%
	);
	background-color: rgb(209, 88, 24);
	border-radius: 30px;
	color: #fff;
	cursor: pointer;
	transition: all 0.5s;
	&:hover {
		background-color: #97b738;
		transform: scale(1.25);
	}
}
img {
	width: 80%;
}
</style>
