<script>
	import { paginate, LightPaginationNav } from 'svelte-paginate';
	export let items = [];
	// let items = filterAbsen;
	let currentPage = 1;
	let pageSize = 15;
	$: paginatedAbsensi = paginate({ items, pageSize, currentPage });
</script>

<table class="table table-striped" id="tabel">
	<thead class="bg-table text-white">
		<tr>
			<th>No</th>
			<th>Nama</th>
			<th>Kelas</th>
			<th>Tanggal</th>
			<th>Status</th>
			<th>Keterangan</th>
		</tr>
	</thead>
	<tbody>
		{#each paginatedAbsensi as absensi, i}
			<tr>
				<td>{absensi.id}</td>
				<td>{absensi.nama}</td>
				<td>{absensi.kelas}</td>
				<td>{absensi.tanggal}</td>
				<td>{absensi.status}</td>
				<td>{absensi.keterangan}</td>
			</tr>
		{/each}
	</tbody>
</table>

<LightPaginationNav
	class="nav"
	totalItems={items.length}
	{pageSize}
	{currentPage}
	limit={1}
	showStepOptions={true}
	on:setPage={(e) => (currentPage = e.detail.page)}
/>

<style>
	#tabel {
		font-size: 12px;
	}
	.bg-table {
		background-color: #4e1ec5;
	}
</style>
