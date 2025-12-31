<script setup lang="ts"></script>

<template>
	<div class="flex flex-col gap-y-6">
		<section class="bg-white/10 p-3 rounded" v-html="$t('help.tutorial.intro')">
		</section>

		<section>
			<div
				class="grid grid-cols-1 xl:grid-cols-[60%_auto] gap-3 gap-x-10">
				<div>
					<h2 class="text-lg font-bold mb-3">{{ $t("help.tutorial.getting_started.title") }}</h2>
					<ol class="list-decimal space-y-2 pl-4">
						<li v-html="$t('help.tutorial.getting_started.step1')"></li>
						<li v-html="$t('help.tutorial.getting_started.step2')"></li>
						<li v-html="$t('help.tutorial.getting_started.step3')"></li>
						<li v-html="$t('help.tutorial.getting_started.step4')"></li>
						<li v-html="$t('help.tutorial.getting_started.step5')"></li>
					</ol>
				</div>
				<div class="flex flex-col justify-center gap-y-3">
					<div class="text-end">
						<img
							src="/images/help/management-screenshot.png"
							alt="Management Screenshot"
							class="rounded shadow-lg border border-white/10" />
						<span class="text-xs text-white/60">{{ $t("nav.items.management") }}</span>
					</div>
				</div>
			</div>
		</section>

		<section>
			<div class="grid grid-cols-1 xl:grid-cols-[30%_auto] gap-3 gap-x-6">
				<div>
					<div class="bg-white/10 rounded p-3">
						<h2 class="text-lg font-bold mb-3">{{ $t("help.tutorial.concepts.title") }}</h2>
						<div class="space-y-6">
							<div>
								<h3 class="text-lg mb-1">{{ $t("help.tutorial.concepts.plans.title") }}</h3>
								<p class="text-white/80">
									{{ $t("help.tutorial.concepts.plans.desc") }}
								</p>
							</div>
							<div>
								<h3 class="text-lg mb-1">{{ $t("help.tutorial.concepts.empires.title") }}</h3>
								<p class="text-white/80">
									{{ $t("help.tutorial.concepts.empires.desc") }}
								</p>
							</div>
							<div>
								<h3 class="text-lg mb-1">{{ $t("help.tutorial.concepts.exchanges.title") }}</h3>
								<p class="text-white/80">
									{{ $t("help.tutorial.concepts.exchanges.desc") }}
								</p>
							</div>
						</div>
					</div>
				</div>
				<div class="p-3">
					<h2 class="text-lg font-bold mb-3">{{ $t("help.tutorial.features.title") }}</h2>
					<div class="space-y-6">
						<div>
							<h3 class="text-lg">{{ $t("help.tutorial.features.base_simulation.title") }}</h3>
							<p class="text-white/80">
								{{ $t("help.tutorial.features.base_simulation.desc") }}
							</p>
						</div>
						<div>
							<h3 class="text-lg">{{ $t("help.tutorial.features.empire_planning.title") }}</h3>
							<p class="text-white/80">
								{{ $t("help.tutorial.features.empire_planning.desc") }}
							</p>
						</div>
						<div>
							<h3 class="text-lg">{{ $t("help.tutorial.features.game_data.title") }}</h3>
							<p class="text-white/80">
								{{ $t("help.tutorial.features.game_data.desc") }}
							</p>
						</div>
						<div>
							<h3 class="text-lg">{{ $t("help.tutorial.features.burn_repair.title") }}</h3>
							<p class="text-white/80">
								{{ $t("help.tutorial.features.burn_repair.desc") }}
							</p>
						</div>
						<div>
							<h3 class="text-lg">
								{{ $t("help.tutorial.features.additional_tools.title") }}
							</h3>
							<p class="text-white/80">
								{{ $t("help.tutorial.features.additional_tools.desc") }}
							</p>
						</div>
					</div>
				</div>
			</div>
		</section>
	</div>
</template>
