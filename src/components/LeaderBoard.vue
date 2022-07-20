<template>
	<div class="leader-board">
		<div class="leader-board-header">
			<img
				src="https://see.fontimg.com/api/renderfont4/mLZ3a/eyJyIjoiZnMiLCJoIjoxNywidyI6MTAwMCwiZnMiOjE3LCJmZ2MiOiIjMDI0MTA2IiwiYmdjIjoiI0ZGRkZGRiIsInQiOjF9/TGVhZGVyIEJvYXJk/terasong.png"
				alt=""
			/>
		</div>
		<div class="table-list">
			<ul>
				<li class="table heading">
					<div>Player</div>
					<div>Score</div>
					<div>Time</div>
				</li>
				<li class="table" v-for="(item, index) in players" :key="index">
					<div v-if="index == 0" class="table-img">
						<img src="https://img.icons8.com/officel/20/000000/medal2.png" />{{
							item.name
						}}
					</div>
					<div v-if="index == 1">
						<img
							src="https://img.icons8.com/officel/20/000000/medal-second-place.png"
						/>{{ item.name }}
					</div>
					<div v-if="index == 2">
						<img
							src="https://img.icons8.com/officel/20/000000/medal2-third-place.png"
						/>{{ item.name }}
					</div>
					<div v-if="index > 2">{{ item.name }}</div>
					<div>{{ item.score }}</div>
					<div>{{ item.timeElapsed || 0 }}</div>
				</li>
			</ul>
		</div>
	</div>
</template>

<script lang="ts">
import { defineComponent } from "vue";
export default defineComponent({
	props: ["players"],
});
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
ul {
	margin: 0;
	padding: 10px 5px;
	border-radius: 5px;
}
.table-list {
	border-radius: 5px;
	background-color: #33cc00;
}
.leader-board-header{
	margin-bottom: 20px;
}
.leader-board {
	margin: 5px;
	padding: 5px;
	font-size: 12px;
}
.table-img {
	img {
		margin-left: 0px;
	}
}
.table {
	display: grid;
	grid-template-columns: 3fr 3fr 2fr;
	max-width: 320px;
	padding: 10px;
	font-style: italic;
	font-size: 15px;

	&.heading {
		font-weight: 600;
	}
}
</style>
