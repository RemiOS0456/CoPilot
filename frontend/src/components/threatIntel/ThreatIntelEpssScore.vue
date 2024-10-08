<template>
	<n-spin :show="loading" content-class="min-h-32">
		<div class="mb-4">
			<KVCard>
				<template #key>cve</template>
				<template #value>{{ cve }}</template>
			</KVCard>
		</div>
		<div class="flex flex-col gap-3" v-if="epssList.length">
			<n-card
				content-class="bg-secondary-color"
				class="overflow-hidden item-appear item-appear-bottom item-appear-005"
				v-for="item of epssList"
				:key="item.___id"
			>
				<div class="flex justify-between gap-8 xs:!flex-row flex-col">
					<n-statistic class="grow" label="Date">
						<span class="stats-value whitespace-nowrap">
							{{ formatDate(item.date, dFormats.date) }}
						</span>
					</n-statistic>
					<n-statistic class="grow" label="EPSS">
						<span class="stats-value">
							{{ item.epss }}
						</span>
					</n-statistic>
					<n-statistic class="grow" label="Percentile">
						<span class="stats-value">
							{{ item.percentile }}
						</span>
					</n-statistic>
				</div>
			</n-card>
			<p v-if="epssModelLink" class="w-full text-right">
				EPSS Model:
				<a :href="epssModelLink" target="_blank">{{ epssModelLink }}</a>
			</p>
		</div>
		<template v-else>
			<n-empty description="No items found" class="justify-center h-48" v-if="!loading" />
		</template>
	</n-spin>
</template>

<script setup lang="ts">
import { ref, onBeforeMount } from "vue"
import { useMessage, NSpin, NEmpty, NCard, NStatistic } from "naive-ui"
import Api from "@/api"
import KVCard from "@/components/common/KVCard.vue"
import { useSettingsStore } from "@/stores/settings"
import { formatDate } from "@/utils"
import { nanoid } from "nanoid"
import type { EpssScore } from "@/types/threatIntel.d"

interface EpssScoreExt extends EpssScore {
	___id?: string
}

const { cve } = defineProps<{
	cve: string
}>()

const dFormats = useSettingsStore().dateFormat
const message = useMessage()
const loading = ref(false)
const epssList = ref<EpssScoreExt[]>([])
const epssModelLink = ref<string | null>(null)

function getData() {
	loading.value = true

	Api.threatIntel
		.epssScore(cve)
		.then(res => {
			if (res.data.success) {
				epssList.value = ((res.data.data as EpssScoreExt[]) || []).map(o => {
					o.___id = nanoid()
					return o
				})
				epssModelLink.value = res.data.the_epss_model
			} else {
				message.warning(res.data?.message || "An error occurred. Please try again later.")
			}
		})
		.catch(err => {
			message.error(err.response?.data?.message || "An error occurred. Please try again later.")
		})
		.finally(() => {
			loading.value = false
		})
}

onBeforeMount(() => {
	getData()
})
</script>

<style lang="scss" scoped>
.stats-value {
	font-family: var(--font-family-mono);
	font-size: clamp(18px, 2.3vw, var(--n-value-font-size));
}
</style>
