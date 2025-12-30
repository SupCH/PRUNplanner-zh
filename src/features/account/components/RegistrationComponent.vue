<script setup lang="ts">
	import { onMounted, ref, Ref, computed, ComputedRef } from "vue";
	import { useI18n } from "vue-i18n";
	const { t } = useI18n();

	import { PForm, PFormItem, PFormSeperator, PButton, PInput } from "@/ui";
	import { IUserRegistrationPayload } from "@/features/api/userData.types";
	import { useQuery } from "@/lib/query_cache/useQuery";

	const isLoading = ref(false);
	const hasError = ref(false);
	const hasErrorMessage: Ref<string | null> = ref(null);
	const registrationSuccess = ref(false);
	const registrationUsername: Ref<string | null> = ref(null);

	const inputUsername: Ref<string | null> = ref(null);
	const inputPassword: Ref<string | null> = ref(null);
	const inputEmail: Ref<string | null> = ref(null);
	const inputPlanetName: Ref<string | null> = ref(null);
	const activeSecurityOption: Ref<string | null> = ref(null);

	const securityOptionList = [
		"OT-580b",
		"KW-688c",
		"ZV-759c",
		"ZV-896b",
		"FK-794b",
		"UV-351c",
		"RC-040b",
		"OT-442b",
		"KW-020c",
	];

	function randomSecurityOption() {
		activeSecurityOption.value =
			securityOptionList[
				Math.floor(Math.random() * securityOptionList.length)
			];
	}

	const canRegister = computed(() => {
		// username at least 3 characters, no spaces
		if (
			inputUsername.value === null ||
			inputUsername.value.length < 3 ||
			inputUsername.value.includes(" ")
		)
			return false;

		// password at least 8 characters
		if (inputPassword.value === null || inputPassword.value.length < 8)
			return false;

		// planetname must be filled
		if (inputPlanetName.value === null || inputPlanetName.value === "")
			return false;

		return true;
	});

	const registrationPayload: ComputedRef<IUserRegistrationPayload> = computed(
		() => ({
			username: inputUsername.value ?? "",
			password: inputPassword.value ?? "",
			planet: inputPlanetName.value ?? "",
			randomplanet: activeSecurityOption.value ?? "",
			...(inputEmail.value ? { email: inputEmail.value } : {}),
		})
	);

	async function registerUser(): Promise<void> {
		isLoading.value = true;
		hasError.value = false;
		registrationSuccess.value = false;
		hasErrorMessage.value = null;

		try {
			const data = await useQuery(
				"PostUserRegistration",
				registrationPayload.value
			).execute();
			registrationUsername.value = data.username;
			registrationSuccess.value = true;
			// eslint-disable-next-line @typescript-eslint/no-explicit-any
		} catch (err: any) {
			// error message is string
			const match = err.message.match(/{.*}$/);
			if (match) {
				hasErrorMessage.value = JSON.parse(match[0]).detail;
			}
			hasError.value = true;
			randomSecurityOption();
		} finally {
			isLoading.value = false;
		}
	}

	onMounted(() => randomSecurityOption());
</script>

<template>
	<div class="mx-auto max-w-[400px]">
		<template v-if="registrationSuccess">
			<div>
				<div class="text-xl text-white font-bold font-mono pb-1">
					{{ $t("auth.register_success_welcome", { username: registrationUsername }) }}
				</div>
				<div class="pt-3">
					{{ $t("auth.register_success_desc") }}
				</div>
			</div>
		</template>
		<template v-else>
			<div class="text-xl text-white font-bold font-mono pb-1">
				{{ $t("auth.account_info") }}
			</div>
			<div class="pb-3 text-white/60 text-xs font-mono">
				{{ $t("auth.register_tos_desc") }}
				<router-link
					to="/imprint-tos"
					class="underline hover:text-link-primary">
					{{ $t("auth.tos_link") }}
				</router-link>
			</div>
			<div v-if="hasError" class="pb-3 text-red-600">
				{{ $t("auth.error_register") }}
				{{ hasErrorMessage }}
			</div>
			<PForm>
				<PFormItem :label="$t('auth.username')">
					<PInput v-model:value="inputUsername" class="w-full" />
					<template #info>
						{{ $t("auth.username_info") }}
					</template>
				</PFormItem>
				<PFormItem :label="$t('auth.password')">
					<PInput
						v-model:value="inputPassword"
						type="password"
						class="w-full" />
					<template #info>
						{{ $t("auth.password_info") }}
					</template>
				</PFormItem>
				<PFormItem :label="$t('auth.email')">
					<PInput
						v-model:value="inputEmail"
						:placeholder="$t('auth.email_placeholder')"
						class="w-full" />
					<template #info>
						{{ $t("auth.email_info") }}
					</template>
				</PFormItem>
				<PFormSeperator>
					<div
						class="text-xl text-white font-bold font-mono pt-3 pb-1">
						{{ $t("auth.security_question") }}
					</div>
					<div class="font-mono text-xs text-white/60 pb-3">
						{{ $t("auth.security_question_desc", { planet: activeSecurityOption, command: 'PLI ' + activeSecurityOption }) }}
					</div>
				</PFormSeperator>
				<PFormItem :label="$t('auth.planet_name_label')">
					<PInput v-model:value="inputPlanetName" class="w-full" />
				</PFormItem>
				<PFormItem label="">
					<PButton
						:disabled="!canRegister"
						:loading="isLoading"
						class="mt-3"
						@click="registerUser">
						{{ $t("auth.register_button") }}
					</PButton>
				</PFormItem>
			</PForm>
		</template>
	</div>
</template>
