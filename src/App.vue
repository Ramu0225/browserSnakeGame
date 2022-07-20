<template>
	<WelcomePage />
</template>

<script lang="ts">
import { defineComponent } from "vue";
import WelcomePage from "./components/Home.vue";

export default defineComponent({
	name: "App",
	components: {
		WelcomePage,
	},
});
</script>

<style lang="scss">
* {
	margin: 0;
	padding: 0;
}
#app {
	font-family: Avenir, Helvetica, Arial, sans-serif;
	-webkit-font-smoothing: antialiased;
	-moz-osx-font-smoothing: grayscale;
	text-align: center;

	width: 100%;
	height: 100vh;
	display: flex;
	justify-content: center;
	align-items: center;
	background-color: #0f1214;
	color: #0f1214;
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
		transform: scale(1.09);
	}
}
</style>
