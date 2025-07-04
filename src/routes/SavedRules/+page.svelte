<script lang="ts">
	import {
		Button,
		Input,
		Modal,
		Navbar,
		NavBrand,
		NavHamburger,
		Select,
		Table,
		TableBody,
		TableBodyCell,
		TableBodyRow,
		TableHead,
		TableHeadCell
	} from 'flowbite-svelte';
	import { user_id, VITE_API_URL } from '$lib/constants';
	import { SideBar } from '../../store/toogleModal.svelte';
	import { onMount } from 'svelte';

	let { data } = $props();
	console.log('data: ', data);

	let currentTab = $state('Insertion');
	let isFormOpen = $state(false); // Modal closed by default
	let isLoading = $state(false);
	let rules = $state(data?.rules?.rules || []);
	let selectedRule = $state(null); // Store the selected rule for modal

	const fetchFiles = async () => {
		try {
			isLoading = true;
			const response = await fetch(`${VITE_API_URL}/rules_book_debt/get_all_rules/${user_id}`);

			if (!response.ok) {
				console.log('Response not OK: ', response.status, response.statusText);
				rules = [];
				console.log(response);
			}

			const result = await response.json();
			if (result.status !== 'success') {
				console.error('API Error: ', result.details);
				rules = [];
				isLoading = false;
				return;
			}
			rules = result.rules;
			isLoading = false;
		} catch (error) {
			console.error('error: ', error);
			isLoading = false;
		}
	};

	onMount(() => {
		fetchFiles();
	});

	const deleteRule = async (ruleId: string) => {
		try {
			isLoading = true;
			const response = await fetch(`${VITE_API_URL}/rules_book_debt/delete_rule/${ruleId}`, {
				method: 'DELETE'
			});

			if (!response.ok) {
				console.error('Delete API Error: ', response.status, response.statusText);
				isLoading = false;
				return;
			}

			const result = await response.json();
			if (result.status !== 'success') {
				console.error('Delete API Error: ', result.details);
				isLoading = false;
				return;
			}

			// Refresh the rules list after successful deletion
			await fetchFiles();
		} catch (error) {
			console.error('Delete error: ', error);
			isLoading = false;
		}
	};

	const filteredRules = $derived(
		rules.filter((rule) => rule.type_of_rule === currentTab.toLowerCase())
	);

	// Function to open modal with selected rule
	const openRuleModal = (rule) => {
		selectedRule = rule;
		isFormOpen = true;
	};
</script>

<Modal
	size="md"
	class={`${currentTab === 'Insertion' ? 'bg-[#E9FFEF]' : 'bg-[#FFEDED]'}`}
	bind:open={isFormOpen}
>
	<div class="flex w-full flex-col gap-6 p-2">
		<div class="w-full flex-col items-center justify-center">
			<span class="flex items-center justify-center text-xl font-semibold text-[#000000DE]">
				{selectedRule?.rule_name || 'No Rule Selected'}
			</span>
			<span class="flex items-center justify-center font-medium text-[#00000080]">
				Pinned to: {selectedRule?.pin ? selectedRule.tag_name : 'None'}
			</span>
		</div>
		{#if selectedRule?.rules?.[0]}
			{#each selectedRule.rules[0] as rule, index}
				<div class="flex flex-col gap-4">
					<span class="flex items-center gap-6">
						<h3>{'IF'}</h3>
						<Input
							type="text"
							value={rule.column}
							disabled
							class="cursor-not-allowed bg-gray-100"
						/>
					</span>
					<span class="flex items-center gap-6">
						<Select value={rule.operator} disabled class="cursor-not-allowed bg-gray-100">
							<option value="equal to">equal to</option>
							<option value="less than">less than</option>
							<option value="greater than">greater than</option>
							<option value="includes">includes</option>
						</Select>
						<Input type="text" value={rule.value} disabled class="cursor-not-allowed bg-gray-100" />
					</span>
					<span class="flex w-full items-center justify-center">{rule.connector}</span>
					{#if rule.then}
						<span class="flex items-center gap-6">
							<Select value={rule.then} disabled class="cursor-not-allowed bg-gray-100">
								<option value="accept">accept</option>
								<option value="reject">reject</option>
							</Select>
						</span>
					{/if}
				</div>
			{/each}
		{:else}
			<p class="text-center text-gray-500">No rules defined</p>
		{/if}
	</div>
</Modal>

<div class="absolute w-full px-8 py-4">
	<Navbar>
		<NavBrand class="flex gap-4 px-6">
			<button aria-label="sidebar button" onclick={() => (SideBar.isSideBarVisible = false)}>
				<svg
					class="h-6 w-6 text-gray-800 dark:text-white"
					aria-hidden="true"
					xmlns="http://www.w3.org/2000/svg"
					width="24"
					height="24"
					fill="none"
					viewBox="0 0 24 24"
				>
					<path
						stroke="currentColor"
						stroke-linecap="round"
						stroke-linejoin="round"
						stroke-width="2"
						d="m7 10 1.99994 1.9999-1.99994 2M12 5v14M5 4h14c.5523 0 1 .44772 1 1v14c0 .5523-.4477 1-1 1H5c-.55228 0-1-.4477-1-1V5c0-.55228.44772-1 1-1Z"
					/>
				</svg>
			</button>
			<button onclick={() => (currentTab = 'Insertion')}>
				<h1
					class="text-2xl font-bold"
					class:text-black={currentTab === 'Insertion'}
					class:text-[#0000005e]={currentTab !== 'Insertion'}
				>
					Insertion Rules
				</h1>
			</button>
			<button onclick={() => (currentTab = 'Ejection')}>
				<h1
					class="cursor-pointer text-2xl font-bold"
					class:text-black={currentTab === 'Ejection'}
					class:text-[#0000005e]={currentTab !== 'Ejection'}
				>
					Ejection Rules
				</h1>
			</button>
		</NavBrand>
	</Navbar>
</div>

<div class="flex h-[100vh] w-full flex-col items-center justify-center pt-12">
	<div class="flex h-[85%] w-[90%] items-center justify-center rounded-3xl">
		{#if isLoading}
			<div class="flex items-center justify-center">
				<div
					class="h-8 w-8 animate-spin rounded-full border-4 border-blue-500 border-t-transparent"
				></div>
				<span class="ml-2">Loading...</span>
			</div>
		{:else if filteredRules && filteredRules.length > 0}
			<div
				class="h-full w-full overflow-y-scroll [-ms-overflow-style:none] [scrollbar-width:none] [&::-webkit-scrollbar]:hidden"
			>
				<Table divClass={'w-full'} class="border-separate border-spacing-y-2">
					<TableHead class="bg-transparent">
						<TableHeadCell class="w-[300px]">Rule Name</TableHeadCell>
						<TableHeadCell>Pinned To</TableHeadCell>
						<TableHeadCell></TableHeadCell>
						<TableHeadCell></TableHeadCell>
						<TableHeadCell></TableHeadCell>
						<TableHeadCell></TableHeadCell>
						<TableHeadCell></TableHeadCell>
					</TableHead>
					<TableBody tableBodyClass="divide-y">
						{#each filteredRules as rule}
							<TableBodyRow class="bg-[#F9FAFB]">
								<TableBodyCell class="max-w-[300px] truncate rounded-md" title={rule.rule_name}>
									{rule.rule_name}
								</TableBodyCell>
								<TableBodyCell>{rule.pin ? rule.tag_name : 'None'}</TableBodyCell>
								<TableBodyCell></TableBodyCell>
								<TableBodyCell></TableBodyCell>
								<TableBodyCell></TableBodyCell>
								<TableBodyCell>
									<button
										aria-label="view details"
										class="cursor-pointer text-[#2362EB]"
										onclick={() => openRuleModal(rule)}
									>
										View Details
									</button>
								</TableBodyCell>
								<TableBodyCell class="rounded-md">
									<button
										aria-label="Delete"
										class="cursor-pointer text-[#2362EB]"
										onclick={() => deleteRule(rule._id)}
									>
										Delete
									</button>
								</TableBodyCell>
							</TableBodyRow>
						{/each}
					</TableBody>
				</Table>
			</div>
		{:else}
			<p class="opacity-50">No {currentTab} Rules Saved</p>
		{/if}
	</div>
</div>

<style>
	.button-group {
		cursor: pointer;
	}
</style>
