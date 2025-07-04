<script lang="ts">
	export let eventName: string;
	export let location: string;
	export let date: string; // ISO-format
	export let displayDate: string; // Lesbar dato

	function downloadICS() {
		// Forventet format: 2025-08-16T18:00:00
		const start = date.replace(/[-:]/g, '').slice(0, 15) + 'Z';
		const endDate = new Date(date);
		endDate.setHours(endDate.getHours() + 2);
		const end = endDate.toISOString().replace(/[-:]/g, '').slice(0, 15) + 'Z';

		const icsContent = [
			'BEGIN:VCALENDAR',
			'VERSION:2.0',
			'BEGIN:VEVENT',
			`SUMMARY:${eventName}`,
			`DTSTART:${start}`,
			`DTEND:${end}`,
			`LOCATION:${location}`,
			'DESCRIPTION:Legg til i kalender',
			'END:VEVENT',
			'END:VCALENDAR'
		].join('\r\n');

		const blob = new Blob([icsContent], { type: 'text/calendar' });
		const url = URL.createObjectURL(blob);

		const a = document.createElement('a');
		a.href = url;
		a.download = `${eventName}.ics`;
		document.body.appendChild(a);
		a.click();
		document.body.removeChild(a);
		URL.revokeObjectURL(url);
	}
</script>

<header class="text-center py-4 mb-4 border-bottom">
	<h1 class="display-4">{eventName}</h1>
	<p class="lead text-muted">
		📍
		<a
			href={`https://www.google.com/maps/search/?api=1&query=${encodeURIComponent(location)}`}
			target="_blank"
			rel="noopener noreferrer"
		>
			{location}
		</a>
		&nbsp;&bull;&nbsp; 🗓️
		<a href="#" on:click|preventDefault={downloadICS} title="Last ned kalenderfil">
			{displayDate}
		</a>
	</p>
</header>
