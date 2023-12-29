<script>
	import axios from 'axios';
	import { beforeUpdate, onMount, onDestroy } from 'svelte';
	import DigitalClock from '../lib/DigitalClock.svelte';
	import DateDisplay from '../lib/DateDisplay.svelte';
	import TableAbsen from '../lib/TableAbsen.svelte';

	//  variabel yang dibutuhkan
	let dataSiswa = [];
	let dataAbsensi = [];
	let focus = '';
	let cari = '';
	let filteredSiswa = [];
	let filterAbsen = [];
	let alert = '';
	let state = {
		id_siswa: '',
		nama: '',
		kelas: '',
		foto: ''
	};
	// api endpoint wa gateway
	const waApiUrl = import.meta.env.VITE_WA_API_URL;
	const dbApiUrl = import.meta.env.VITE_DB_API_URL;

	// Pengaturan Waktu

	let currentTime = new Date();

	let hours = currentTime.getHours();
	let minutes = currentTime.getMinutes();

	let targetHours = 7;
	let targetMinutes = 30;
	let targetMasukHours = 8;
	let targetMasukMinutes = 30;
	// menegecek status masuk dan pulang
	$: statusMasuk = hours <= targetHours && minutes <= targetMinutes ? 'tepat waktu' : 'terlambat';
	$: statusPulang = hours <= 12 && hours <= 13 ? 'Pulang Lebih Cepat' : 'Pulang Tepat Waktu';
	// otomatisasi status absen
	$: statusAbsen =
		hours <= targetMasukHours
			? 'Masuk'
			: hours <= 8 && hours <= 12
			? 'AbsenNonAktif'
			: hours >= 12 && hours < 16
			? 'Pulang'
			: 'AbsenNonAktif';

	// ambil data siswa
	const getDataSiswa = async () => {
		const response = await axios.get(`${dbApiUrl}/data-siswa`);
		dataSiswa = response.data;
		filterAbsen = dataAbsensi.filter((absen) => {
			return absen.tanggal == date;
		});
	};
	// ambil data absensi
	const getAbsensi = async () => {
		const response = await axios.get(`${dbApiUrl}/absensi`);
		dataAbsensi = response.data;
	};

	// ambil data dan susun hanya untuk ini hari
	beforeUpdate(() => {
		getAbsensi();
		// postPesan();
		filterAbsen = dataAbsensi.filter((absen) => {
			return absen.tanggal == date;
		});
	});

	// ketika halaman di load
	onMount(() => {
		getDataSiswa();
		focus.focus();
		// getAbsensi();
	});
	// ambil tanggal
	let date = new Date().toLocaleDateString('id-ID', {
		year: 'numeric',
		month: 'long',
		day: 'numeric'
	});
	// bunyi bel bisa diganti dari file lokal
	function bunyi() {
		var bel = new Audio('https://www.meramukoding.com/wp-content/uploads/2020/05/doorbell.mp3');
		bel.play();
	}
	// fungsi scan barcode dan kirim data serta kirim pesan ke wa
	const searching = () => {
		filteredSiswa = dataSiswa.filter((siswa) => {
			return siswa.id_siswa === cari;
		});
		state = {
			id_siswa: filteredSiswa[0].id_siswa,
			nama: filteredSiswa[0].nama,
			tanggal: date,
			status: statusAbsen == 'Masuk' ? statusMasuk : statusPulang,
			keterangan: statusAbsen,
			kelas: filteredSiswa[0].kelas,
			hp: filteredSiswa[0].hp,
			foto: filteredSiswa[0].foto
		};

		if (filteredSiswa && cari != '') {
			bunyi();
			console.log(state);
		} else {
			alert = 'data tidak ditemukan';
		}
		axios.post(`${dbApiUrl}/absensi`, state);
		axios.post(waApiUrl, {
			number: state.hp,
			message: `Halo Pak/Bu,  saya ${
				state.nama
			}  sudah absen ${statusAbsen} hari ini tanggal: ${date} dengan status *${
				statusAbsen == 'Masuk' ? statusMasuk : statusPulang
			}* `
		});

		alert = 'Absensi Berhasil';

		setInterval(() => {
			cari = '';
			filteredSiswa = [];
			alert = '';
			focus.focus();
		}, 2000);
	};

	$: nonAktif = statusAbsen == 'AbsenNonAktif' ? 'd-none' : '';

	// Function to reload the page
	function reloadPage() {
		window.location.reload();
		alert('halo');
	}

	// Function to check and reload the page at 12:00 PM and 4:10 PM
	function checkAndReload() {
		const now = new Date();
		const hours = now.getHours();
		const minutes = now.getMinutes();

		// Check if it's 12:00 PM
		if (hours == 12 && minutes == 3) {
			reloadPage();
		}
		if (hours == 16 && minutes == 10) {
			reloadPage();
		}
	}

	// Use onMount to start the interval when the component is mounted
	onMount(() => {
		// Check and reload every minute
		const intervalId = setInterval(checkAndReload, 30000);

		// Use onDestroy to clear the interval when the component is destroyed
		onDestroy(() => {
			clearInterval(intervalId);
		});
	});
	const refresh = () => {
		window.location.reload();
	};
</script>

<div class="container contain">
	<div class="row justify-content-center">
		<div class="col-md-6 text-center shadow rounded-lg">
			<h3 class="text-header">
				Sistem Absensi Digital dengan <span class="text-span"> Barcode dan WA Gateway </span>
			</h3>
			<!--  -->
			<DigitalClock />
			<!-- <p>{hours}</p> -->

			<span class="btn btn-danger">{statusAbsen}</span>
			<div class="mt-4">
				<input
					bind:value={cari}
					bind:this={focus}
					on:input={searching}
					type="number"
					class="form-control-lg {nonAktif}"
				/>
				{#if alert}
					<div class="alert alert-success" role="alert">Absensi Berhasil</div>
				{/if}
			</div>
			<div>
				{#if filteredSiswa.length == 0 && cari != ''}
					<p>data tidak ditemukan</p>
				{:else}
					{#each filteredSiswa as siswa}
						<div>
							<p>{siswa.id_siswa}</p>
							<p>{siswa.nama}</p>
							<p>{siswa.kelas}</p>
							<p>{siswa.foto}</p>
						</div>
					{/each}
				{/if}
			</div>
		</div>
		<div class="col-md-6 shadow rounded-lg">
			<div class="row d-flex justify-content-between align-middle">
				<div class="col-md-3">
					<DateDisplay />
				</div>
				<div class="col-md-3">
					<button class="btn btn-success btn-sm" on:click={refresh}>Refresh</button>
				</div>
			</div>	

			<h3 class="text-center text-header">Data Absesnsi</h3>

			<TableAbsen items={filterAbsen} />
		</div>
	</div>
</div>

<style>
	.text-header {
		font-size: 1.6rem;
		font-weight: bold;
		color: #4e1ec5;
	}
	.text-span {
		color: #d41a20;
	}
	/* * {
		border: 1px solid red;
	} */
</style>
