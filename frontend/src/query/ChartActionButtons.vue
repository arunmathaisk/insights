<script setup>
import PublicShareDialog from '@/components/PublicShareDialog.vue'
import useDashboards from '@/dashboard/useDashboards'
import { downloadImage } from '@/utils'
import { Maximize } from 'lucide-vue-next'
import { computed, inject, ref, watch } from 'vue'

const emit = defineEmits(['fullscreen'])
const query = inject('query')

const showShareDialog = ref(false)
const showDashboardDialog = ref(false)
const dashboards = useDashboards()
const toDashboard = ref(null)
const addingToDashboard = ref(false)
const dashboardOptions = computed(() => {
	return dashboards.list.map((d) => {
		return { label: d.title, value: d.name }
	})
})

const $notify = inject('$notify')
function onAddToDashboard() {
	if (query.chart.doc.chart_type == 'Auto') {
		$notify({
			variant: 'warning',
			title: 'Choose a chart type',
			message: "Chart type cannot be 'Auto' to add to dashboard",
		})
		return
	}
	showDashboardDialog.value = true
}
const addChartToDashboard = async () => {
	if (!toDashboard.value) return
	await query.chart.addToDashboard(toDashboard.value.value)
	showDashboardDialog.value = false
	$notify({
		variant: 'success',
		title: 'Success',
		message: 'Chart added to dashboard',
	})
}

watch(showDashboardDialog, (val) => val && dashboards.reload(), { immediate: true })
</script>

<template>
	<div class="flex gap-2">
		<Button variant="outline" @click="emit('fullscreen')">
			<template #icon> <Maximize class="h-4 w-4" /> </template>
		</Button>
		<Button variant="outline" @click="onAddToDashboard()"> Add to Dashboard </Button>
		<Button variant="outline" @click="showShareDialog = true"> Share </Button>
	</div>

	<PublicShareDialog
		v-if="query.chart.doc?.doctype && query.chart.doc?.name"
		v-model:show="showShareDialog"
		:resource-type="query.chart.doc.doctype"
		:resource-name="query.chart.doc.name"
		:allow-public-access="true"
		:isPublic="Boolean(query.chart.doc.is_public)"
		@togglePublicAccess="query.chart.togglePublicAccess"
	/>

	<Dialog :options="{ title: 'Add to Dashboard' }" v-model="showDashboardDialog">
		<template #body-content>
			<div class="text-base">
				<span class="mb-2 block text-sm leading-4 text-gray-700">Dashboard</span>
				<Autocomplete :options="dashboardOptions" v-model="toDashboard" />
			</div>
		</template>
		<template #actions>
			<Button variant="solid" @click="addChartToDashboard" :loading="addingToDashboard">
				Add
			</Button>
		</template>
	</Dialog>
</template>
