<script setup>
import BaseChart from '@/components/Charts/BaseChart.vue'
import { computed } from 'vue'
import getBarChartOptions from './getBarChartOptions'

const props = defineProps({
	data: { type: Object, required: true },
	options: { type: Object, required: true },
})

const labels = computed(() => {
	if (!props.data?.length || !props.options.xAxis) return []
	return props.data.map((d) => d[props.options.xAxis])
})

const datasets = computed(() => {
	if (!props.data?.length || !props.options.yAxis) return []
	const yAxis =
		typeof props.options.yAxis === 'string' ? [props.options.yAxis] : props.options.yAxis
	return (
		yAxis
			// to exclude the columns that might be removed from the query but not the chart
			.filter(
				(series) =>
					props.data[0].hasOwnProperty(series?.column) ||
					props.data[0].hasOwnProperty(series)
			)
			.map((series) => {
				const column = series.column || series
				return {
					label: column,
					data: props.data.map((d) => d[column]),
					series_options: series.series_options || {},
				}
			})
	)
})

const barChartOptions = computed(() => {
	return getBarChartOptions(labels.value, datasets.value, props.options)
})
</script>

<template>
	<BaseChart :title="props.options.title" :options="barChartOptions" />
</template>
