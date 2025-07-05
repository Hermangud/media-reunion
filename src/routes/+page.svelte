<script lang="ts">
	import Header from '$lib/components/Header.svelte';
	import RegistrationForm from '$lib/components/RegistrationForm.svelte';
	import PaymentInfo from '$lib/components/PaymentInfo.svelte';
	import StatusChecker from '$lib/components/StatusChecker.svelte';
	import ImageUpload from '$lib/components/ImageUpload.svelte';

	let registrationComplete = $state(false);
	let registeredKid = $state<number | null>(null);

	function handleRegistrationSuccess(kid: number) {
		registrationComplete = true;
		registeredKid = kid;
	}

	// Definer informasjonen om arrangementet her
	const eventDetails = {
		name: 'Media Reunion 2025',
		location: 'Mindre-alvs vei 11, 1672 Kråkerøy',
		date: '2025-08-16T15:00:00', // ISO-format for ICS
		displayDate: 'Lørdag 16. august 2025, kl. 17:00' // For visning
	};
</script>

<svelte:head>
	<title>{eventDetails.name} - Påmelding</title>
	<meta
		name="description"
		content="Meld deg på til {eventDetails.name}. Her finner du all informasjon du trenger."
	/>
</svelte:head>

<Header
	eventName={eventDetails.name}
	location={eventDetails.location}
	date={eventDetails.date}
	displayDate={eventDetails.displayDate}
/>

{#if registrationComplete}
	<div class="alert alert-success mt-4" role="alert">
		<h4 class="alert-heading">Takk for din påmelding!</h4>
		<p>
			Vi har mottatt dine detaljer. Vennligst fullfør betalingen nedenfor for å sikre din plass.
		</p>
	</div>
	<PaymentInfo kidNumber={registeredKid} />
{:else}
	<main>
		<section id="registration">
			<h2 class="mb-3">Meld deg på her</h2>
			<RegistrationForm onSuccess={handleRegistrationSuccess} />
		</section>
	</main>
{/if}
<hr class="my-5" />
<ImageUpload />
<hr class="my-5" />
<section id="status-check" class="mb-5">
	<h2 class="mb-3">Sjekk din påmeldingsstatus</h2>
	<StatusChecker />
</section>
