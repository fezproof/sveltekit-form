<script lang="ts">
	import { invalidate } from '$app/navigation';

	import { page } from '$app/stores';
	import type { HttpMethod, JSONObject } from '@sveltejs/kit/types/private';
	import { createEventDispatcher } from 'svelte';

	export let action: string = $page.url.href;
	export let method: HttpMethod = 'post';

	let state: 'idle' | 'submitting' | 'invalidating' = 'idle';
	let result: JSONObject | undefined = undefined;
	let submitted: FormData | undefined = undefined;

	const dispatch = createEventDispatcher<{
		submit: { submitted: FormData; form: HTMLFormElement };
		success: { data: JSONObject | undefined; form: HTMLFormElement };
		complete: { form: HTMLFormElement };
	}>();

	const submit: (
		event: SubmitEvent & { currentTarget: EventTarget & HTMLFormElement }
	) => Promise<unknown> = async (event) => {
		console.log('Submitted...');
		state = 'submitting';

		const form = event.currentTarget;

		const body = new FormData(form);
		submitted = body;
		dispatch('submit', { submitted: body, form });

		try {
			const res = await fetch(form.action, {
				method: form.method,
				headers: {
					accept: 'application/json'
				},
				body
			});

			if (!res.ok) {
				throw new Error(await res.text());
			}

			result = await res.json();
			dispatch('success', { data: result, form });

			state = 'invalidating';
			await invalidate(action);
			dispatch('complete', { form });

			state = 'idle';

			return;
		} catch (e: unknown) {
			// maybe handle this better
			throw e;
		}
	};
</script>

<form
	class={$$props.class}
	{method}
	on:submit|preventDefault={submit}
	action={action ?? $page.url.href}
>
	<slot {state} {result} {submitted} />
</form>
