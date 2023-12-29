<template>
	<view>
		<button class="c-button" :class="isColor ? 'yesColor' : 'noColor'" @touchstart="touch" :style="buttonStyle">
			<template v-if="isLoading">
				<view style="display: flex; align-items: center; justify-content: center;">
					<image style="width: 25rpx; height: 25rpx;" src="@/static/user-loading.gif" mode=""></image>
				</view>
			</template>
			<template v-else>
				<slot></slot>
			</template>
			<view v-if="isRipple" class="waveRipple" :class="[isWaving ? 'waveActive' : '']" :style="rippleStyle">
			</view>
		</button>
	</view>
</template>

<script setup lang="ts">
	import { ref, computed, watch, onMounted, nextTick, getCurrentInstance } from 'vue';
	const instance = getCurrentInstance();

	// 定义props
	const props = defineProps({
		width: {
			type: String,
			default: uni.getSystemInfoSync().screenWidth * 0.9 + 'px'
		},
		height: {
			type: String,
			default: '94rpx'
		},
		color: {
			type: String,
			default: '#4DC3FF'
		},
		isColor: {
			type: Boolean,
			default: true
		},
		isRipple: {
			type: Boolean,
			default: true
		},
		radius: {
			type: String,
			default: '22px'
		},
		isLoading: {
			type: Boolean,
			default: false
		}
	});

	// 定义data
	const rippleToTop = ref(0);
	const rippleToLeft = ref(0);
	const domInfo = ref<{ targetWidth ?: string }>({});
	const isWaving = ref(false);
	let flag = ref(false);

	// 按钮样式
	const buttonStyle = computed(() => ({
		width: props.width,
		height: props.height,
		borderRadius: props.radius,
		backgroundColor: props.color
	}));

	// 水波样式
	const rippleStyle = computed(() => ({
		top: rippleToTop.value + 'px',
		left: rippleToLeft.value + 'px',
		width: domInfo.value.targetWidth + 'px',
		height: domInfo.value.targetWidth + 'px',
		backgroundColor: 'rgba(0, 0, 0, 0.15)'
	}));

	const throttle = (fc, waitTime = 500, imme = true) => {
		if (imme) {
			if (!flag.value) {
				flag.value = true;
				typeof fc === 'function' && fc();
				setTimeout(() => {
					flag.value = false;
				}, waitTime);
			}
		} else {
			if (!flag.value) {
				flag.value = true;
				setTimeout(() => {
					flag.value = false;
					typeof fc === 'function' && fc();
				}, waitTime);
			}
		}
	};

	const touch = (e) => {
		if (!props.isColor || props.isLoading) return;
		// emit('change');
		throttle(() => {
			isWaving.value = false;
			nextTick(() => {
				getWaveQuery(e);
			});
		});
	};

	const getWaveQuery = async (e) => {
		const res = await getElQuery();
		let data = res[0];
		if (!data.width || !data.height) return;
		data.targetWidth = data.height > data.width ? data.height : data.width;
		if (!data.targetWidth) return;
		domInfo.value = data;
		// 根据不同平台获取touches的位置
		let touchesX = e.touches[0].clientX || null;
		let touchesY = e.touches[0].clientY || null;
		rippleToTop.value = touchesY - data.top - data.targetWidth / 2;
		rippleToLeft.value = touchesX - data.left - data.targetWidth / 2;
		nextTick(() => {
			isWaving.value = true;
		});
	};

	const getElQuery = () => {
		return new Promise(resolve => {
			let queryInfo = uni.createSelectorQuery().in(instance.proxy);
			//#ifdef MP-ALIPAY
			queryInfo = uni.createSelectorQuery();
			//#endif
			queryInfo.select('.c-button').boundingClientRect();
			queryInfo.exec(data => {
				resolve(data);
			});
		});
	};

	// 生命周期钩子
	onMounted(() => {
		// 在组件挂载后执行的代码
	});

	// 监听
	watch(() => props.isLoading, (newVal, oldVal) => {
		// isLoading发生变化时执行的代码
	});
</script>


<style lang="scss" scoped>
	.c-button {
		display: flex;
		align-items: center;
		justify-content: center;
		position: relative;
		overflow: hidden;
		line-height: 1;
		z-index: 1;
		box-sizing: border-box;
		transition: all 0.12s;
		white-space: nowrap;
	}

	.c-button::after {
		border: none;
		transform-origin: center;
		transform: scale(1.5);
	}

	.waveRipple {
		z-index: 0;
		position: absolute;
		border-radius: 100%;
		background-clip: padding-box;
		transform-origin: center;
		pointer-events: none;
		transform: scale(0);
		user-select: none;
		opacity: 1;
	}

	.waveRipple.waveActive {
		opacity: 0;
		transform: scale(2);
		transition: opacity 0.6s linear, transform 0.6s linear;
	}

	.yesColor {
		animation-name: yesColorAnimation;
		animation-fill-mode: forwards;
		animation-duration: 800ms;
	}

	.noColor {
		animation-name: noColorAnimation;
		animation-fill-mode: forwards;
		animation-duration: 800ms;
	}

	@keyframes yesColorAnimation {
		0% {
			background: #D2D2D2;
		}

		100% {
			// background: linear-gradient(229deg, #FD4251 0%, #FC5B32 100%);
			background-color: #4DC3FF;

		}
	}

	@keyframes noColorAnimation {
		0% {
			// background: linear-gradient(229deg, #FD4251 0%, #FC5B32 100%);
			background-color: #4DC3FF;
		}

		100% {
			background: #D2D2D2;
		}
	}
</style>