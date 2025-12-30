<script setup lang="ts">
	import { computed } from "vue";
	import { useI18n } from "vue-i18n";
	const { t } = useI18n();

	const screenshots = computed(() => [
		{
			image: "/images/homepage/homepage-empire.png",
			text: t("home.screenshots.empire"),
		},
		{
			image: "/images/homepage/homepage-plan.png",
			text: t("home.screenshots.plan"),
		},
		{
			image: "/images/homepage/homepage-management.png",
			text: t("home.screenshots.management"),
		},
		{
			image: "/images/homepage/homepage-exchanges.png",
			text: t("home.screenshots.exchanges"),
		},
		{
			image: "/images/homepage/homepage-market-exploration.png",
			text: t("home.screenshots.exploration"),
		},
		{
			image: "/images/homepage/homepage-planet-search.png",
			text: t("home.screenshots.search"),
		},
	]);
</script>

<template>
	<div class="mx-auto w-full max-w-7xl pb-5 md:px-10 lg:px-0">
		<div class="lg:mb-10">
			<h1
				class="bg-gradient-to-b bg-clip-text pb-4 text-7xl font-extrabold leading-tight text-transparent from-white/50 to-white">
				{{ $t("home.title") }} <br />
				{{ $t("home.subtitle") }}
			</h1>
			<p class="text-xl text-white/60 font-mono">
				{{ $t("home.description") }}
			</p>
		</div>

		<h1 class="pb-4 text-4xl tracking-tight text-white text-center mt-30">
			{{ $t("home.features_title") }}
		</h1>
		<p class="text-white/60 text-center text-xl mb-10">
			{{ $t("home.features_subtitle") }}
		</p>

		<div
			class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 bg-white/20 gap-[1px] child:p-10 child:hover:bg-linear-315 child:from-white/10 child:to-transparent mb-20">
			<div class="bg-black">
				<div class="flex flex-col">
					<div class="text-white text-2xl font-bold mb-2">
						{{ $t("home.features.base_simulation_title") }}
					</div>
					<div class="text-white/60 text-lg font-mono">
						{{ $t("home.features.base_simulation_desc") }}
					</div>
				</div>
			</div>
			<div class="bg-black">
				<div class="flex flex-col">
					<div class="text-white text-2xl font-bold mb-2">
						{{ $t("home.features.empire_planning_title") }}
					</div>
					<div class="text-white/60 text-lg font-mono">
						{{ $t("home.features.empire_planning_desc") }}
					</div>
				</div>
			</div>
			<div class="bg-black">
				<div class="flex flex-col">
					<div class="text-white text-2xl font-bold mb-2">
						{{ $t("home.features.game_data_title") }}
					</div>
					<div class="text-white/60 text-lg font-mono">
						{{ $t("home.features.game_data_desc") }}
					</div>
				</div>
			</div>
			<div class="bg-black">
				<div class="flex flex-col">
					<div class="text-white text-2xl font-bold mb-2">
						{{ $t("home.features.burn_repair_title") }}
					</div>
					<div class="text-white/60 text-lg font-mono">
						{{ $t("home.features.burn_repair_desc") }}
					</div>
				</div>
			</div>
			<div class="bg-black">
				<div class="flex flex-col">
					<div class="text-white text-2xl font-bold mb-2">
						{{ $t("home.features.market_exploration_title") }}
					</div>
					<div class="text-white/60 text-lg font-mono">
						{{ $t("home.features.market_exploration_desc") }}
					</div>
				</div>
			</div>
			<div class="bg-black">
				<div class="flex flex-col">
					<div class="text-white text-2xl font-bold mb-2">
						{{ $t("home.features.additional_tools_title") }}
					</div>
					<div class="text-white/60 text-lg font-mono">
						{{ $t("home.features.additional_tools_desc") }}
					</div>
				</div>
			</div>
		</div>

		<div>
			<h1
				class="pb-4 text-4xl tracking-tight text-white text-center mt-30">
				{{ $t("home.command_anywhere_title") }}
			</h1>
			<p class="text-white/60 text-center text-xl mb-10">
				{{ $t("home.command_anywhere_subtitle") }}
			</p>
			<div class="grid grid-cols-1 xl:grid-cols-3 gap-3">
				<div
					v-for="(screenshot, index) in screenshots"
					:key="screenshot.image"
					:class="[index >= 3 ? 'hidden xl:block' : '']">
					<img :src="screenshot.image" :alt="screenshot.text" />
					<div class="text-center font-mono text-white/60 px-2">
						{{ screenshot.text }}
					</div>
				</div>
			</div>
		</div>
	</div>

	<div class="mx-auto w-full max-w-7xl px-5 md:px-10 lg:px-0">
		<h1 class="pb-4 text-4xl tracking-tight text-white text-center mt-30">
			{{ $t("home.core_concepts_title") }}
		</h1>
		<p class="text-white/60 text-center text-xl mb-10">
			{{ $t("home.core_concepts_subtitle") }}
		</p>
		<div
			class="grid grid-cols-1 lg:grid-cols-3 divide-x divide-white/20 child:p-10 mb-10">
			<div>
				<div class="text-white text-2xl font-bold mb-2">{{ $t("home.concepts.plans_title") }}</div>
				<div class="text-lg font-mono text-white/60">
					{{ $t("home.concepts.plans_desc") }}
				</div>
			</div>
			<div>
				<div class="text-white text-2xl font-bold mb-2">{{ $t("home.concepts.empires_title") }}</div>
				<div class="text-lg font-mono text-white/60">
					{{ $t("home.concepts.empires_desc") }}
				</div>
			</div>
			<div class="">
				<div class="text-white text-2xl font-bold mb-2">{{ $t("home.concepts.exchanges_title") }}</div>
				<div class="text-lg font-mono text-white/60">
					{{ $t("home.concepts.exchanges_desc") }}
				</div>
			</div>
		</div>
	</div>
	<div>
		<h1 class="pb-4 text-4xl tracking-tight text-white text-center mt-20">
			{{ $t("home.powered_by") }}
		</h1>
		<div class="flex flex-row items-center justify-center pt-3 gap-10">
			<a href="https://posthog.com/" target="_blank">
				<img
					src="/images/homepage/posthog-logo-white@2x.png"
					class="h-[30px]" />
			</a>
			<a href="https://www.highcharts.com/" target="_blank">
				<img
					src="/images/homepage/highcharts-logo.png"
					class="h-[30px]" />
			</a>
		</div>
	</div>
	<h1 class="pb-4 text-4xl tracking-tight text-white text-center mt-30">
		{{ $t("home.open_source_title") }}
	</h1>
	<p class="text-white/60 text-center text-xl mb-5">
		{{ $t("home.open_source_desc") }}
	</p>
	<p class="text-white/60 text-center font-mono">
		{{ $t("home.made_with") }}
	</p>
</template>
