<script setup lang="ts">
	import { ref, Ref, computed } from "vue";
	import { useI18n } from "vue-i18n";
	const { t } = useI18n();

	import { useUserStore } from "@/stores/userStore";

	const userStore = useUserStore();
	import router from "@/router";

	// UI
	import { PForm, PFormItem, PFormSeperator, PInput, PButton } from "@/ui";

	const inputUsername: Ref<string | null> = ref(null);
	const inputPassword: Ref<string | null> = ref(null);
	const isLoggingIn: Ref<boolean> = ref(false);
	const hasError: Ref<boolean> = ref(false);

	const canLogin = computed(() => {
		if (inputUsername.value === null || inputUsername.value.length < 3)
			return false;
		if (inputPassword.value === null || inputPassword.value.length < 8)
			return false;

		return true;
	});

	async function handleLogin(): Promise<void> {
		if (!canLogin.value) return;

		isLoggingIn.value = true;
		hasError.value = false;

		const result = await userStore.performLogin(
			inputUsername.value!,
			inputPassword.value!
		);

		isLoggingIn.value = false;

		if (!result) hasError.value = true;
		else {
			router.push({ path: "/empire/" });
		}
	}
</script>

<template>
	<div class="mx-auto max-w-[400px]">
		<div class="text-xl text-white font-bold font-mono pb-3">{{ $t("auth.login_title") }}</div>
		<div v-if="hasError" class="pb-3 text-red-600">
			{{ $t("auth.error_login") }}
		</div>
		<PForm>
			<PFormSeperator>
				<div class="font-mono text-xs text-white/60 pb-3">
					{{ $t("auth.tos_prefix") }}
					<router-link
						to="/imprint-tos"
						class="hover:cursor-pointer underline">
						{{ $t("auth.tos_link") }}
					</router-link>
				</div>
			</PFormSeperator>
			<PFormItem :label="$t('auth.username')">
				<PInput v-model:value="inputUsername" class="w-full" />
			</PFormItem>
			<PFormItem :label="$t('auth.password')">
				<PInput
					v-model:value="inputPassword"
					type="password"
					class="w-full" />
			</PFormItem>
			<PFormSeperator>
				<div class="font-mono text-xs text-white/60 py-3">
					{{ $t("auth.forgot_password_prefix") }}
					<router-link
						to="/request-password-reset"
						class="hover:cursor-pointer underline">
						{{ $t("auth.forgot_password_link") }}
					</router-link>
				</div>
			</PFormSeperator>
			<PFormItem label="">
				<PButton
					:loading="isLoggingIn"
					:disabled="!canLogin"
					@click="handleLogin">
					{{ $t("auth.login_title") }}
				</PButton>
			</PFormItem>
		</PForm>
	</div>
</template>
