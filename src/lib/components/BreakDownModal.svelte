<script lang="ts">
	import {
		Drawer,
		Button,
		CloseButton,
		Sidebar,
		SidebarGroup,
		SidebarWrapper,
		Timeline,
		TimelineItem
	} from 'flowbite-svelte';
	import { CalendarWeekSolid } from 'flowbite-svelte-icons';
	import { sineIn } from 'svelte/easing';
	import {
		breakDownBar,
		breakdownTransactionID,
		transactionModal
	} from '../../store/toogleModal.svelte';
	import { VITE_API_URL } from '$lib/constants';

	let transitionParams = {
		x: 320,
		duration: 200,
		easing: sineIn
	};

	// Reactive data array
	let data = $state([]);
	let summary = $state({});

	let isLoading = $state(false);

	// Use $effect to react to changes in the store
	$effect(() => {
		if (!breakDownBar.isBreakDownBarVisible && breakdownTransactionID.transaction_id) {
			fetchData(breakdownTransactionID.transaction_id);
		}
	});

	async function fetchData(transaction_id: string) {
		try {
			isLoading = true;
			const response = await fetch(
				`${VITE_API_URL}/transaction/get_all_data_for_one_transaction/${transaction_id}`,
				{
					method: 'GET',
					headers: {
						'Content-Type': 'application/json'
						// Add any authentication headers if required
					}
				}
			);

			if (!response.ok) {
				throw new Error('Failed to fetch transaction data');
			}

			const result = await response.json();
			// Assuming the API returns an array of steps or similar structure
			data = result.breakdown || []; // Adjust based on your API response structure
			summary = result.summary || {};
			console.log(data);
		} catch (error) {
			console.error('Error fetching transaction data:', error);
			data = []; // Reset data on error
		} finally {
			isLoading = false;
		}
	}

	const downloadFile = async (file_path: string) => {
		try {
			const downloadUrl = `${VITE_API_URL}/project/download_file?file_path=${encodeURIComponent(file_path)}`;
			console.log('Download URL:', downloadUrl);

			const response = await fetch(downloadUrl, {
				method: 'GET',
				headers: {
					// Add any authentication headers if needed
				}
			});

			if (!response.ok) {
				throw new Error(`Download failed: ${response.statusText}`);
			}

			const blob = await response.blob();
			const blobUrl = window.URL.createObjectURL(blob);
			const filename = file_path.split('/').pop() || 'download.xlsx';
			const a = document.createElement('a');
			a.href = blobUrl;
			a.download = filename;
			a.style.display = 'none';
			document.body.appendChild(a);
			a.click();
			document.body.removeChild(a);
			window.URL.revokeObjectURL(blobUrl);
			console.log('Download completed successfully');
		} catch (error) {
			console.error('Error downloading file:', error);
			alert('Failed to download file. Please try again.');
		}
	};
</script>

<Drawer
	transitionType="fly"
	{transitionParams}
	bind:hidden={breakDownBar.isBreakDownBarVisible}
	id="sidebar3"
	position="absolute"
	class="right-0 start-[1p] w-[30%]"
>
	<div class="flex items-center">
		<h5
			id="drawer-navigation-label-3"
			class="text-base font-semibold text-black dark:text-gray-400"
		>
			<span>BreakDown</span>
		</h5>
		<!-- <span class="bg-[#F5F5F5] px-4 py-2">{file_name}</span> -->
		<CloseButton
			onclick={() => {
				breakDownBar.isBreakDownBarVisible = true;
				breakdownTransactionID.transaction_id = null;
			}}
			class="mb-4 dark:text-white"
		/>
	</div>
	<Sidebar>
		<SidebarWrapper divClass="overflow-y-auto py-4 px-3 rounded-sm dark:bg-gray-800">
			<SidebarGroup>
				<Timeline order="vertical">
					{#if isLoading}
						<div class="absolute inset-0 z-30 flex h-full w-full items-center justify-center">
							<div
								class="z-40 h-8 w-8 animate-spin rounded-full border-4 border-blue-500 border-t-transparent"
							></div>
						</div>
					{:else}
						{#each data as step}
							<TimelineItem title={step.step_name} date={step.date}>
								{#snippet orientationSlot()}
									<span
										class="bg-primary-200 dark:bg-primary-900 absolute -start-3 flex h-6 w-6 items-center justify-center rounded-full ring-8 ring-white dark:ring-gray-900"
									>
										<CalendarWeekSolid class="text-primary-600 dark:text-primary-400 h-4 w-4" />
									</span>
								{/snippet}
								<p class="mb-2 text-base font-normal text-gray-500 dark:text-gray-400">
									{step.description}
								</p>
								<div class="flex items-center gap-4 text-sm font-semibold text-blue-700">
									<button
										onclick={() => downloadFile(summary.base_file_path)}
										class="cursor-pointer">Original File</button
									>
									<button onclick={() => downloadFile(step.file_path)} class="cursor-pointer"
										>Final File</button
									>
								</div>
							</TimelineItem>
						{:else}
							<p class="text-gray-500">No data available</p>
						{/each}
					{/if}
				</Timeline>
			</SidebarGroup>
		</SidebarWrapper>
	</Sidebar>
</Drawer>
