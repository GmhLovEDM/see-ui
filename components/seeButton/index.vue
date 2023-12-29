<template>
	<view>
		<button class="c-button" :class="isColor ? 'yesColor' : 'noColor'" @touchstart="touch" :style="{
			 	width: width,
			 	height: height,
				borderRadius: radius,
			 	backgroundColor: color}">
			<template v-if="isLoading">
				<view style="display: flex;align-items: center;justify-content: center;">
					<image style="width: 25rpx;height: 25rpx;"
						src="@/static/user-loading.gif"
						mode=""></image>
				</view>
			</template>
			<template v-else>
				<slot></slot>
			</template>
			<view v-if="isRipple" class="waveRipple" :class="[isWaving?'waveActive':'']" :style="{
					top: rippleToTop + 'px',
					left: rippleToLeft + 'px',
					width: domInfo.targetWidth + 'px',
					height: domInfo.targetWidth + 'px',
					backgroundColor: 'rgba(0, 0, 0, 0.15)'}">
			</view>
		</button>
	</view>
</template>

<script>
	export default {
		name: "c-button",
		props: {
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
		},
		data() {
			return {
				rippleToTop: 0,
				rippleToLeft: 0,
				domInfo: {},
				isWaving: false,
				timer: {},
				flag: false,
			};
		},
		created() {

		},
		methods: {
			touch(e) {
				if (!this.isColor) return;
				if (this.isLoading) return;
				this.$emit('change')
				this.throttle(() => {
					this.isWaving = false
					this.$nextTick(function() {
						this.getWaveQuery(e)
					})
				})
			},
			throttle(fc, waitTime = 500, imme = true) {
				if (imme) {
					if (!this.flag) {
						this.flag = true
						typeof fc === 'function' && fc()
						this.timer = setTimeout(() => {
							this.flag = false
						}, waitTime)
					}
				} else {
					if (!this.flag) {
						this.flag = true
						this.timer = setTimeout(() => {
							this.flag = false
							typeof fc === 'function' && fc()
						}, waitTime)
					}
				}
			},
			getWaveQuery(e) {
				this.getElQuery().then(res => {
					let data = res[0]
					if (!data.width || !data.width) return
					data.targetWidth = data.height > data.width ? data.height : data.width
					if (!data.targetWidth) return
					this.domInfo = data;
					let touchesX = ''
					let touchesY = ''
					// #ifndef MP-BAIDU || MP-ALIPAY
					touchesX = e.touches[0].clientX
					touchesY = e.touches[0].clientY
					// #endif
					// #ifdef MP-BAIDU
					touchesX = e.changedTouches[0].clientX
					touchesY = e.changedTouches[0].clientY
					// #endif
					// #ifdef MP-ALIPAY
					touchesX = e.detail.clientX
					touchesY = e.detail.clientY
					// #endif
					this.rippleToTop = touchesY - data.top - data.targetWidth / 2
					this.rippleToLeft = touchesX - data.left - data.targetWidth / 2
					this.$nextTick(() => {
						this.isWaving = true
					})
				})
			},
			getElQuery() {
				return new Promise(resolve => {
					let queryInfo = ''
					queryInfo = uni.createSelectorQuery().in(this)
					//#ifdef MP-ALIPAY
					queryInfo = uni.createSelectorQuery()
					//#endif
					queryInfo.select('.c-button').boundingClientRect()
					queryInfo.exec(data => {
						resolve(data)
					})
				})
			},
		}
	}
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