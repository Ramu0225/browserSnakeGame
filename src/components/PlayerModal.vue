<template>
	<transition name="modal">
		<div v-show="modalActive" class="modal-mask">
			<transition name="modal-inner">
				<div v-show="modalActive" class="modal-container">
					<!-- Modal Content -->
					<slot />
					<div v-if="gameOverModal">
						<button @click="restart" type="button">Play Again</button>
						<button @click="restartGameOnSwitchingUser" type="button">
							Switch User
						</button>
					</div>
					<div v-else>
						<button @click="close" type="button" :disabled="buttonDisabled">
							Play
						</button>
					</div>
				</div>
			</transition>
		</div>
	</transition>
</template>

<script lang="ts">
import { defineComponent } from "vue";
export default defineComponent({
	name: "PlayerModal",
	props: {
		modalActive: Boolean,
		buttonDisabled: Boolean,
		gameOverModal: Boolean,
	},
	setup(props, { emit }) {
		const close = () => {
			emit("close");
		};
		const restart = () => {
			emit("restart");
		};
		const restartGameOnSwitchingUser = () => {
			emit("restartGameOnSwitchingUser");
		};
		return { close, restart, restartGameOnSwitchingUser };
	},
});
</script>

<style lang="scss" scoped>
.modal-enter-active,
.modal-leave-active {
	transition: opacity 0.3s;
}
.modal-enter-from,
.modal-leave-to {
	opacity: 0;
}
.modal-inner-enter-active {
	transition: all 0.3s;
}
.modal-inner-leave-active {
	transition: all 0.3s;
}
.modal-inner-enter-from {
	opacity: 0;
	transform: scale(0.8);
}
.modal-inner-leave-to {
	transform: scale(0.8);
}
.modal-mask {
	display: flex;
	justify-content: center;
	align-items: center;
	height: 100vh;
	width: 100vw;
	position: fixed;
	top: 0;
	left: 0;
	background-color: rgba(255, 255, 255, 0.7);
	.modal-container {
		display: flex;
		justify-content: space-around;
		align-items: center;
		flex-direction: row;
		width: 400px;
		border-radius: 5px;
		background-color: #fff;
		padding: 10px;
		button {
			padding: 10px 20px;
			border: none;
			background-color: rgb(4, 8, 32);
			border-radius: 5px;
			color: #fff;
			cursor: pointer;
		}
		button[disabled] {
			pointer-events: none;
			background-color: rgb(145, 150, 181);
		}
	}
}
</style>
