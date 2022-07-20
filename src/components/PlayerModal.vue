<template>
	<transition name="modal">
		<div v-show="modalActive" class="modal-mask">
			<transition name="modal-inner">
				<div v-show="modalActive" class="modal-container">
					<!-- Modal Content -->
					<slot />
					<div v-if="gameOverModal">
						<button @click="restart" class="btn-play" type="button">
							Play Again
						</button>
						<button
							@click="restartGameOnSwitchingUser"
							class="btn-play"
							type="button"
						>
							Switch User
						</button>
					</div>
					<div v-else>
						<button
							@click="close"
							class="btn-play"
							type="button"
							:disabled="buttonDisabled"
						>
							Play!
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
	transition: all 0.1s;
}
.modal-inner-leave-active {
	transition: all 0.1s;
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
	height: 550px;
	width: 525px;
	position: fixed;
	top: offset;
	left: offset;
	background-color: rgba(255, 255, 255, 0.7);
	.modal-container {
		display: flex;
		justify-content: space-between;
		align-items: center;
		flex-direction: column;
		width: 300px;
		border-radius: 5px;
		//background-color: rgb(224, 108, 13);
		background-color: #33cc00;
		border: 2px solid rgb(224, 108, 13);
		padding: 10px;
	}
	.btn-play {
		margin: 10px;
		color: rgb(39, 93, 39);
	}
	button[disabled] {
		pointer-events: none;
	}
}
</style>
