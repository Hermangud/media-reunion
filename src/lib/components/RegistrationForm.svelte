<script lang="ts">
	import { supabase } from '$lib/supabaseClient';

	let { onSuccess }: { onSuccess: (kid: number) => void } = $props();
	let isLoading = $state<boolean>(false);
	let feedbackMessage = $state<string>('');
	let messageType = $state<'success' | 'error'>('success');

	async function handleFormSubmit(event: SubmitEvent): Promise<void> {
		event.preventDefault();
		isLoading = true;
		feedbackMessage = '';
		const form = event.currentTarget as HTMLFormElement;

		try {
			const formData = new FormData(form);
			const name = formData.get('name') as string;
			const email = formData.get('email') as string;

			const { data: newKid, error: rpcError } = await supabase.rpc('register_guest_and_get_kid', {
				guest_name: name,
				guest_email: email
			});
			if (rpcError) throw rpcError;

			messageType = 'success';
			feedbackMessage = 'Takk, din påmelding er mottatt!';

			form.reset();

			if (onSuccess) {
				onSuccess(newKid);
			}
		} catch (error: any) {
			messageType = 'error';
			if (error.message.includes('duplicate key')) {
				feedbackMessage = 'Denne e-posten er allerede registrert.';
			} else {
				feedbackMessage = 'En feil oppstod. Prøv igjen.';
			}
			console.error('Feil i påmelding:', error);
		} finally {
			isLoading = false;
		}
	}
</script>

<form onsubmit={handleFormSubmit}>
	<div class="mb-3">
		<label for="name" class="form-label">Fullt navn</label>
		<input type="text" id="name" name="name" class="form-control" required disabled={isLoading} />
	</div>

	<div class="mb-3">
		<label for="email" class="form-label">E-postadresse</label>
		<input
			type="email"
			id="email"
			name="email"
			class="form-control"
			required
			disabled={isLoading}
		/>
	</div>

	<div class="d-grid">
		<button type="submit" class="btn btn-primary btn-lg" disabled={isLoading}>
			{#if isLoading}
				<span class="spinner-border spinner-border-sm" role="status" aria-hidden="true"></span>
				Laster opp...
			{:else}
				Meld meg på
			{/if}
		</button>
	</div>
</form>

{#if feedbackMessage}
	<div
		class="alert mt-4 {messageType === 'success' ? 'alert-success' : 'alert-danger'}"
		role="alert"
	>
		{feedbackMessage}
	</div>
{/if}
