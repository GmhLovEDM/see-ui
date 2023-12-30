<template>
	<button class="see-button" :style="buttonStyle" @tap="tapClick">
		<slot></slot>
	</button>
</template>

<script lang="ts" setup>
	import { computed } from 'vue'

	const props = withDefaults(defineProps<{
		width : string | number;
		height : string | number;
		type : 'primary' | 'success' | 'warning' | 'error'
	}>(), {
		width: '88px',
		height: "44px",
	});
	const emits = defineEmits<{
		(e : 'tapButton', event : Event) : void
	}>()

	const buttonStyle = computed(() => ({
		width: props.width,
		height: props.height
	}))

	const tapClick = (e : Event) => {
		emits('tapButton', e)
	}
</script>

<style lang="scss" scoped>
	@import '@/components/index.scss';

	.see-button {
		border: 0;
		border-radius: 0;
	}
</style>