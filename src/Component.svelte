<script>
	import { createEventDispatcher, onDestroy, onMount } from 'svelte';
	import 'ag-grid-community/dist/styles/ag-grid.css';
	import 'ag-grid-community/dist/styles/ag-theme-alpine.css';
	const dispatch = createEventDispatcher();
	export let columnDefs;
	export let data = [];
	export let theme = 'alpine';
	export let filter;
	export let api;
	export let pixel = '200px';
	export let fcolumn = false;
	export let options = {
		defaultColDef: {
			flex: 1,
			minWidth: 150,
			filter: true
		},
		suppressFieldDotNotation: false,
		rowSelection: 'single'
	};
	export let loading = false;
	let themeUrl = `https://unpkg.com/ag-grid-community/dist/styles/ag-theme-${theme}.css`;
	let ref;
	let grid;
	let showing;

	const filterGrid = () => {
		api?.setQuickFilter(filter);
	};
	const onSelectionChanged = () => {
		const selectedRows = api.getSelectedRows();
		dispatch('select', selectedRows);
	};
	const onCellValueChanged = (e) => {
		data[e.rowIndex] = e.data;
		dispatch('update', { row: e.rowIndex, data: e.data });
	};
	const onGridReady = () => {
		api = grid.gridOptions.api;
		if (loading) api.showLoadingOverlay();
	};
	const updateData = (data) => {
		if (grid && api) {
			api.setRowData(data);
			api.setColumnDefs(columnDefs);
		}
	};
	onMount(async () => {
		const { Grid } = await import('ag-grid-community');
		grid = new Grid(ref, {
			...options,
			columnDefs,
			rowData: data,
			onGridReady,
			onCellValueChanged,
			onSelectionChanged
		});
	});
	onDestroy(() => {
		if (grid) {
			grid.destroy();
		}
	});
	$: filterGrid(filter);
	$: updateData(data);
	$: total = data.length;
	$: {
		filter;
		fcolumn;
		showing = api?.getDisplayedRowCount();
	}
</script>

<svelte:head>
	{#if theme !== 'alpine'}
		<link rel="stylesheet" href={themeUrl} />
	{/if}
</svelte:head>
<div class="grid" style="height: calc(100vh - {pixel})">
	<div class="filter">
		<div>
			<slot />
		</div>
		<input type="text" bind:value={filter} placeholder="Quick Filter" />
	</div>
	<div class="grid-content">
		<div bind:this={ref} style="height: 100%; width:100%" class="ag-theme-{theme}" />
	</div>
	<div class="grid-footer">
		{showing} of {total} rows
	</div>
</div>

<style>
	:global(:root) {
		--grid-height: 100%;
	}
	.center-text {
		text-align: center;
	}
	.grid-footer {
		margin-top: 1em;
		display: flex;
		justify-content: flex-end;
	}
	.filter {
		display: flex;
		justify-content: space-between;
	}
	.filter > input {
		padding: 0.5em;
	}
	.grid-content {
		width: 100%;
		height: var(--grid-height, 800px);
	}
</style>
