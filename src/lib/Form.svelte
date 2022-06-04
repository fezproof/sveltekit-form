<script lang="ts">
	import { invalidate } from '$app/navigation';

	import { page } from '$app/stores';
	import type { HttpMethod, JSONObject } from '@sveltejs/kit/types/private';
	import { createEventDispatcher, tick } from 'svelte';

	export let action: string = $page.url.href;
	export let method: HttpMethod = 'post';

	let idle = true;
	let submitting = false;
	let invalidating = false;

	let result: JSONObject | undefined = undefined;
	let submitted: FormData | undefined = undefined;

	const dispatch = createEventDispatcher<{
		submit: { submitted: FormData; form: HTMLFormElement };
		success: { data: JSONObject | undefined; form: HTMLFormElement; submitted: FormData };
		complete: { form: HTMLFormElement; submitted: FormData };
	}>();

	const setFormState = (state: 'idle' | 'submitting' | 'invalidating') => {
		switch (state) {
			case 'idle': {
				idle = true;
				submitting = false;
				invalidating = false;
				break;
			}
			case 'submitting': {
				idle = false;
				submitting = true;
				invalidating = false;
				break;
			}
			case 'invalidating': {
				idle = false;
				submitting = false;
				invalidating = true;
				break;
			}
		}
	};

	const submit: (
		event: SubmitEvent & { currentTarget: EventTarget & HTMLFormElement }
	) => Promise<unknown> = async (event) => {
		setFormState('submitting');

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
			setFormState('invalidating');
			await tick();
			dispatch('success', { data: result, form, submitted: body });

			const url = new URL(form.action);
			await invalidate(url.href);

			setFormState('idle');

			dispatch('complete', { form, submitted: body });
			return;
		} catch (e: unknown) {
			setFormState('idle');
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
	<slot {idle} {submitting} {invalidating} {result} {submitted} />
</form>
