<script lang="ts">
	let {
		eventName,
		location,
		date,
		displayDate
	}: { eventName: string; location: string; date: string; displayDate: string } = $props();

	function downloadICS(event: MouseEvent) {
		event.preventDefault(); // Hindrer lenken i å navigere

		const start = date.replace(/[-:]/g, '').slice(0, 15) + 'Z';
		const endDate = new Date(date);
		endDate.setHours(endDate.getHours() + 4); // Setter festen til å vare i 4 timer
		const end = endDate.toISOString().replace(/[-:]/g, '').slice(0, 15) + 'Z';

		const icsContent = [
			'BEGIN:VCALENDAR',
			'VERSION:2.0',
			'BEGIN:VEVENT',
			`SUMMARY:${eventName}`,
			`DTSTART:${start}`,
			`DTEND:${end}`,
			`LOCATION:${location}`,
			`DESCRIPTION:Se deg på ${eventName}!`,
			'END:VEVENT',
			'END:VCALENDAR'
		].join('\r\n');

		const blob = new Blob([icsContent], { type: 'text/calendar' });
		const url = URL.createObjectURL(blob);
		const a = document.createElement('a');
		a.href = url;
		a.download = `${eventName}.ics`;
		a.click();
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
		&nbsp;•&nbsp; 🗓️
		<a href="#" onclick={downloadICS} title="Legg til i kalender">
			{displayDate}
		</a>
	</p>
</header>
