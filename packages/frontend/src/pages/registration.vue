<!--
SPDX-FileCopyrightText: syuilo and other misskey contributors
SPDX-License-Identifier: AGPL-3.0-only
-->

<template>
<div v-if="meta">
	<XSetup v-if="meta.requireSetup"/>
	<XEntrance v-else/>
</div>
</template>

<script lang="ts" setup>
import { computed } from 'vue';
import XSetup from './welcome.setup.vue';
import XEntrance from './welcome.entrance.a.vue';
import { useRouter } from '@/router.js';
import { instanceName } from '@/config.js';
import * as os from '@/os.js';
import { definePageMetadata } from '@/scripts/page-metadata.js';
import XSignupDialog from '@/components/MkSignupDialog.vue';

const router = useRouter();

let meta = $ref(null);
let code: string = (new URL(`https://msky.summersweet.jp${router.currentPath}`)).searchParams.get('code') ?? '';
console.log(code);

os.api('meta', { detail: true }).then(res => {
	meta = res;
});

const headerActions = $computed(() => []);

const headerTabs = $computed(() => []);

definePageMetadata(computed(() => ({
	title: instanceName,
	icon: null,
})));

try {
	const accessToken = await os.api('discord/token', {
		code: code,
	});
	const ticket = await os.api('discord/check', {
		discordToken: accessToken.access_token,
	});
	os.popup(XSignupDialog, {
		autoSet: true,
		code: ticket.code,
	}, {}, 'closed');
} catch { /* empty */ }
</script>
