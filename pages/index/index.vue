<template>
	<view class="page">
		<SeeButton>{{ title }}</SeeButton>
	</view>
</template>

<script lang="ts" setup>
	import { ref } from "vue";
	import SeeButton from '@/components/seeButton/index.vue'
	const title = ref('Hello World!')
</script>

<style lang="scss" scoped>

</style>