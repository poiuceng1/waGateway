<script>
	let currentDate = new Date();

	// Mendapatkan hari, tanggal, dan tahun
	let dayOfWeek = currentDate.toLocaleString('id-ID', { weekday: 'long' }); // Ganti 'id-ID' dengan kode bahasa yang diinginkan
	let day = currentDate.getDate();
	let month = currentDate.getMonth() + 1; // Ingat bahwa bulan dimulai dari 0 (Januari adalah bulan 0)
	let year = currentDate.getFullYear();
</script>

<h6>Hari <span class="day">{dayOfWeek}</span> tanggal {day}/{month}/{year}</h6>

<style>
	.day {
		font-weight: bold;
		color: brown;
	}
</style>
