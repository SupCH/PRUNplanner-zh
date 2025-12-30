<script setup lang="ts">
	import {
		computed,
		ComputedRef,
		PropType,
		ref,
		Ref,
		WritableComputedRef,
	} from "vue";

	// Types & Interfaces
	import {
		IProductionBuildingRecipe,
		IRecipeBuildingOption,
	} from "@/features/planning/usePlanCalculation.types";

	// Util
	import { trackEvent } from "@/lib/analytics/useAnalytics";
	import { humanizeTimeMs } from "@/util/date";
	import { formatNumber } from "@/util/numbers";

	// Components
	import MaterialTile from "@/features/material_tile/components/MaterialTile.vue";
	import PlanCOGM from "@/features/planning/components/tools/PlanCOGM.vue";

	// UI
	import { PButton, PInputNumber } from "@/ui";
	import { NModal, NTable } from "naive-ui";
	import { ClearSharp, AnalyticsOutlined } from "@vicons/material";

	const props = defineProps({
		disabled: {
			type: Boolean,
			required: true,
		},
		recipeData: {
			type: Object as PropType<IProductionBuildingRecipe>,
			required: true,
		},
		recipeIndex: {
			type: Number,
			required: true,
		},
		recipeOptions: {
			type: Array as PropType<IRecipeBuildingOption[]>,
			required: true,
		},
		cxUuid: {
			type: String,
			required: false,
			default: undefined,
		},
		planetId: {
			type: String,
			required: true,
		},
	});

	const emit = defineEmits<{
		(
			e: "update:building:recipe:amount",
			index: number,
			value: number
		): void;
		(e: "delete:building:recipe", index: number): void;
		(e: "update:building:recipe", index: number, recipeid: string): void;
	}>();

	// Local State
	const localRecipeOptions: ComputedRef<IRecipeBuildingOption[]> = computed(
		() => {
			// sort by output ticker ascending, map-join multiple tickers
			return [...props.recipeOptions].sort((a, b) => {
				const tickerA = a.Outputs.map((o) => o.Ticker)
					.sort()
					.join("#");
				const tickerB = b.Outputs.map((o) => o.Ticker)
					.sort()
					.join("#");
				return tickerA.localeCompare(tickerB);
			});
		}
	);
	const localRecipeIndex: ComputedRef<number> = computed(() =>
		props.recipeIndex.valueOf()
	);

	const localRecipeData: ComputedRef<IProductionBuildingRecipe> = computed(
		() => props.recipeData
	);
	const localRecipeAmount: WritableComputedRef<number> = computed({
		get: () => props.recipeData.amount,
		set: () => {},
	});
	const refShowRecipeOptions: Ref<boolean> = ref(false);
	const refShowCOGM: Ref<boolean> = ref(false);

	const cogmEnabled = computed(
		() => localRecipeData.value.cogm && localRecipeData.value.cogm.visible
	);

	const cogmWithCX = computed(() => !!props.cxUuid);
</script>

<template>
	<n-modal
		:key="`COGM#RECIPE#${recipeData.recipe.BuildingTicker}#${localRecipeIndex}`"
		v-model:show="refShowCOGM"
		preset="card"
		:title="$t('plan.production.recipe.cogm_title')"
		:class="cogmWithCX ? 'max-w-[1000px]' : 'max-w-[600px]'">
		<PlanCOGM
			v-if="localRecipeData.cogm && cxUuid"
			:cogm-data="localRecipeData.cogm"
			:cx-uuid="cxUuid"
			:planet-id="planetId" />
	</n-modal>
	<div class="flex flex-col">
		<div class="pb-1">
			<PInputNumber
				v-model:value="localRecipeAmount"
				:disabled="disabled"
				show-buttons
				:min="0"
				@update:value="
					(value) => {
						if (value !== null && value !== undefined) {
							emit(
								'update:building:recipe:amount',
								recipeIndex,
								value
							);
						}
					}
				" />
		</div>
		<div
			v-click-outside="
				() => {
					refShowRecipeOptions = false;
				}
			"
			class="border border-pp-border p-3 flex flex-row justify-between child:my-auto hover:cursor-pointer"
			@click="refShowRecipeOptions = true">
			<div class="flex flex-col gap-1">
				<MaterialTile
					v-for="material in localRecipeData.recipe.Outputs"
					:key="`${localRecipeData.recipe.BuildingTicker}#${material.Ticker}`"
					:ticker="material.Ticker"
					:amount="material.Amount * localRecipeData.amount" />
			</div>
			<div class="text-white/50 text-xs text-end">
				<span class="font-bold">
					{{ humanizeTimeMs(localRecipeData.time) }}
				</span>
				<br />
				<span
					>{{
						formatNumber(localRecipeData.dailyShare * 100)
					}}
					%</span
				>
			</div>
		</div>
		<div
			class="relative z-10"
			:class="refShowRecipeOptions ? 'visible' : 'hidden'">
			<n-table
				class="absolute w-full border border-pp-border !bg-black"
				striped>
				<thead>
					<tr>
						<th>{{ $t("plan.production.recipe.table.input") }}</th>
						<th>{{ $t("plan.production.recipe.table.time") }}</th>
						<th>{{ $t("plan.production.recipe.table.output") }}</th>
						<th>{{ $t("plan.production.recipe.table.daily_revenue") }}</th>
						<th>{{ $t("plan.production.recipe.table.profit_area") }}</th>
						<th>{{ $t("plan.production.recipe.table.roi") }}</th>
					</tr>
				</thead>
				<tbody>
					<tr
						v-for="(recipe, index) in localRecipeOptions"
						:key="`${recipe.BuildingTicker}#${recipe.RecipeId}`"
						class="child:whitespace-nowrap hover:cursor-pointer"
						@click="
							emit(
								'update:building:recipe',
								localRecipeIndex,
								recipe.RecipeId
							)
						">
						<td
							:class="
								recipe.RecipeId === localRecipeData.recipeId
									? '!border-l-3 !border-l-green-500'
									: ''
							">
							<div class="flex flex-row gap-1">
								<MaterialTile
									v-for="material in recipe.Inputs"
									:key="`${index}#INPUT#${material.Ticker}`"
									:ticker="material.Ticker"
									:amount="material.Amount" />
							</div>
						</td>
						<td>
							{{ humanizeTimeMs(recipe.TimeMs) }}
						</td>
						<td>
							<div class="flex flex-row gap-1">
								<MaterialTile
									v-for="material in recipe.Outputs"
									:key="`${index}#OUTPUT#${material.Ticker}`"
									:ticker="material.Ticker"
									:amount="material.Amount" />
							</div>
						</td>
						<td
							:class="
								recipe.dailyRevenue >= 0
									? '!text-positive'
									: '!text-negative'
							">
							{{ formatNumber(recipe.dailyRevenue) }} $
						</td>
						<td
							:class="
								recipe.profitPerArea >= 0
									? '!text-positive'
									: '!text-negative'
							">
							{{ formatNumber(recipe.profitPerArea) }} $
						</td>
						<td
							:class="
								recipe.roi >= 0
									? '!text-positive'
									: '!text-negative'
							">
							{{ formatNumber(recipe.roi) }} d
						</td>
					</tr>
				</tbody>
				<tbody>
					<tr>
						<td
							colspan="6"
							class="text-xs !p-2 !text-white/60 !border-t-1">
							<strong>{{ $t("plan.production.recipe.table.daily_revenue") }}</strong> {{ $t("plan.production.recipe.info.revenue_day") }} 
							<strong>{{ $t("plan.production.recipe.table.profit_area") }}</strong> {{ $t("plan.production.recipe.info.dollar_area") }} 
							<strong>{{ $t("plan.production.recipe.table.roi") }}</strong> {{ $t("plan.production.recipe.info.roi_payback") }}
						</td>
					</tr>
				</tbody>
			</n-table>
		</div>

		<div class="flex flex-row justify-between pt-1 child:my-auto">
			<PButton
				size="sm"
				:disabled="!cogmEnabled"
				@click="
					() => {
						refShowCOGM = true;
						trackEvent('plan_tool_cogm', {
							planetNaturalId: props.planetId,
							recipeId: localRecipeData.recipeId,
						});
					}
				">
				<template #icon><AnalyticsOutlined /> </template>
			</PButton>
			<PButton
				size="sm"
				type="error"
				@click="
					() => {
						emit('delete:building:recipe', localRecipeIndex);
					}
				">
				<template #icon><ClearSharp /></template>
			</PButton>
		</div>
	</div>
</template>
