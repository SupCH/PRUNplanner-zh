<script setup lang="ts">
	import { computed } from "vue";

	const { selectedKey, selectedEmpireUuid } = defineProps<{
		selectedKey: string | null;
		selectedEmpireUuid: string | null;
	}>();

	// Util
	import { copyToClipboard } from "@/util/data";

	// UI
	import { PButton } from "@/ui";
	import { ContentCopySharp } from "@vicons/material";

	const keyValue = computed(() => (selectedKey ? selectedKey : ""));

	const urlExchangeData = computed(
		() =>
			`http://api.prunplanner.org/csv/exchange?api_key=${keyValue.value}`
	);

	const urlMaterialData = computed(
		() =>
			`http://api.prunplanner.org/csv/material?api_key=${keyValue.value}`
	);

	const urlMaterialIO = computed(() =>
		selectedEmpireUuid === null
			? `http://api.prunplanner.org/csv/user/materialio?api_key=${keyValue.value}`
			: `http://api.prunplanner.org/csv/user/materialio/${selectedEmpireUuid}/?api_key=${keyValue.value}`
	);
</script>

<template>
	<h2 class="text-white/80 font-bold text-lg pb-3">{{ $t("api.endpoints.title") }}</h2>

	<div class="flex flex-col gap-3">
		<div class="border-2 border-white/10 px-2 py-1">
			<h2 class="text-white/80 font-bold pb-1">{{ $t("api.endpoints.exchange.title") }}</h2>
			<div class="text-white/60">
				{{ $t("api.endpoints.exchange.description") }}
			</div>
			<div
				class="px-2 py-1 bg-black border border-white/10 rounded-sm font-mono my-1 flex flex-row justify-between child:my-auto">
				<div>
					{{ urlExchangeData }}
				</div>
				<PButton size="sm" @click="copyToClipboard(urlExchangeData)">
					<template #icon>
						<ContentCopySharp />
					</template>
				</PButton>
			</div>
			<pre
				class="overflow-auto border border-white/10 rounded-sm bg-white/5 px-2 py-1 text-white/60">
TICKER.EXCHANGECODE,TICKER,EXCHANGECODE,ASK,BID,AVG,SUPPLY,DEMAND,TRADED
DW.AI1,DW,AI1,109.0,105.0,109.0,188563.0,134081.0,9437
DCS.PP30D_AI1,DCS,PP30D_AI1,,,1611.05,,,0</pre
			>
		</div>

		<div class="border-2 border-white/10 px-2 py-1">
			<h2 class="text-white/80 font-bold pb-1">{{ $t("api.endpoints.material.title") }}</h2>
			<div class="text-white/60">
				{{ $t("api.endpoints.material.description") }}
			</div>
			<div
				class="px-2 py-1 bg-black border border-white/10 rounded-sm font-mono my-1 flex flex-row justify-between child:my-auto">
				<div>
					{{ urlMaterialData }}
				</div>
				<PButton size="sm" @click="copyToClipboard(urlMaterialData)">
					<template #icon>
						<ContentCopySharp />
					</template>
				</PButton>
			</div>
			<pre
				class="overflow-auto border border-white/10 rounded-sm bg-white/5 px-2 py-1 text-white/60">
TICKER,NAME,MATERIALID,CATEGORYID,CATEGORYNAME,WEIGHT,VOLUME
AAR,antennaArray,d4a247f1ae7a17b6e80057866ecdf90d,19d621bb3f297c0425e34cdf0b138ece,electronic devices,0.7799999713897705,0.5
RAT,rations,83dd61885cf6879ff49fe1419f068f10,3f047ec3043bdd795fd7272d6be98799,consumables (basic),0.20999999344348907,0.10000000149011612</pre
			>
		</div>

		<div class="border-2 border-white/10 px-2 py-1">
			<h2 class="text-white/80 font-bold pb-1">{{ $t("api.endpoints.material_io.title") }}</h2>
			<div class="text-white/60">
				{{ $t("api.endpoints.material_io.description") }}
			</div>
			<div
				class="px-2 py-1 bg-black border border-white/10 rounded-sm font-mono my-1 flex flex-row justify-between child:my-auto">
				<div>
					{{ urlMaterialIO }}
				</div>
				<PButton size="sm" @click="copyToClipboard(urlMaterialIO)">
					<template #icon>
						<ContentCopySharp />
					</template>
				</PButton>
			</div>
			<pre
				class="overflow-auto border border-white/10 rounded-sm bg-white/5 px-2 py-1 text-white/60">
PLANET,PLAN,TICKER,INPUT,OUTPUT,DELTA,STORAGE,BURN
VH-331g,c9bd3e29-48fe-4795-9d5d-b92c661cb4a0,ALE,0.0,210.65625,210.65625,,
ZV-194a,291c8cf5-401c-4f1b-a191-9f6d2df1339d,DW,27.2,0.0,-27.2,745.0,27.389705882352942</pre
			>
		</div>
	</div>
</template>
