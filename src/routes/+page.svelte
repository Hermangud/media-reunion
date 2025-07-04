<script lang="ts">
	import Header from '$lib/components/Header.svelte';
	import RegistrationForm from '$lib/components/RegistrationForm.svelte';
	import PaymentInfo from '$lib/components/PaymentInfo.svelte';
	import StatusChecker from '$lib/components/StatusChecker.svelte';

	// Definer informasjonen om arrangementet her
	const eventDetails = {
		name: 'Media Reunion 2025',
		location: 'Mindre-alvs vei 11, 1672 Kråkerøy',
		date: 'Lørdag 16. august 2025, kl. 18:00'
	};

	// Bruker Svelte 5 Runes for reaktivitet
	let registrationComplete = $state(false);

	// Denne funksjonen sendes som en prop til RegistrationForm
	function handleRegistrationSuccess() {
		registrationComplete = true;
		// Scroller til toppen for å vise suksessmelding
		window.scrollTo({ top: 0, behavior: 'smooth' });
	}
</script>

<svelte:head>
	<title>{eventDetails.name} - Påmelding</title>
	<meta
		name="description"
		content="Meld deg på til {eventDetails.name}. Her finner du all informasjon du trenger."
	/>
</svelte:head>

<Header eventName={eventDetails.name} location={eventDetails.location} date={eventDetails.date} />

{#if registrationComplete}
	<div class="alert alert-success mt-4" role="alert">
		<h4 class="alert-heading">Takk for din påmelding!</h4>
		<p>
			Vi har mottatt dine detaljer. Vennligst fullfør betalingen nedenfor for å sikre din plass.
		</p>
	</div>
	<PaymentInfo />
{:else}
	<main>
		<section id="registration">
			<h2 class="mb-3">Meld deg på her</h2>
			<RegistrationForm on:success={handleRegistrationSuccess} />
		</section>
	</main>
{/if}

<hr class="my-5" />

<section id="status-check" class="mb-5">
	<h2 class="mb-3">Sjekk din påmeldingsstatus</h2>
	<StatusChecker />
</section>
