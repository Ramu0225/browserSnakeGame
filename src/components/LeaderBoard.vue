<template>
	<h1>Leader Board</h1>
	<ul>
		<li class="table heading">
			<div>Player</div>
			<div>Score</div>
			<div>Time</div>
		</li>
		<li class="table" v-for="(item, index) in list" :key="index">
			<div>{{ item.name }}</div>
			<div>{{ item.score }}</div>
			<div>{{ item.timeElapsed || 0 }}</div>
		</li>
	</ul>
</template>

<script lang="ts">
import { defineComponent } from "vue";

export default defineComponent({
	props: ["list"],
});
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
ul {
	margin: 0;
	padding: 10px 5px;
}
h1 {
	font-size: 1em;
}
.table {
	display: grid;
	grid-template-columns: 2fr 1fr 2fr;
	max-width: 250px;
	padding: 10px;
	font-style: italic;
	&.heading {
		font-weight: 600;
	}
}
</style>
