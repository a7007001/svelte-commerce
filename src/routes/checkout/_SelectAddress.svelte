<script>
import { createEventDispatcher } from 'svelte'
import { session } from '$app/stores'
import { goto } from '$app/navigation'
import AddressSkeleton from './_AddressSkeleton.svelte'
import Radio from '$lib/ui/Radio.svelte'
import { onMount } from 'svelte'
import { del, get } from '../../util/api'
import { toast } from './../../util'
const dispatch = createEventDispatcher()

let id = null
export let returnUrl = null
export let addReturnUrl = null
export let selectedAddress = null
let skeleton = false,
	iconloading = false,
	addresses = null
$: user = $session.user

onMount(() => {
	getAddress()
})
function selectFirstAddress(x) {
	selectedAddress = x?.addresses[0]?.id
	return x
}
function addressChanged(e) {
	selectedAddress = e
	dispatch('addressChanged', e)
}
async function getAddress() {
	try {
		skeleton = true
		addresses = await get('addresses/my')
		// Can not push automatically, because it creates bad user experience when user clicks on back to address list page
		// if (addresses.count < 1) goto(addReturnUrl)
		selectedAddress = addresses?.data[0]?.id
	} catch (e) {
	} finally {
		skeleton = false
	}
}
function edit(id) {
	goto(`${returnUrl}?id=${id}`)
}
async function remove(id) {
	if (confirm('Are you sure to delete?')) {
		try {
			iconloading = true
			await del(`addresses/${id}`)
			await getAddress()
			toast('Address deleted successfully', 'success')
		} catch (e) {
			toast(e, 'error')
		} finally {
			iconloading = false
		}
	}
}
</script>

<section>
	{#if skeleton}
		<AddressSkeleton />
	{:else if addresses?.data?.length}
		<div>
			{#each addresses.data as a}
				<div class="border-b py-2 pb-4 sm:py-6">
					<label class="flex flex-row w-full px-2 sm:px-6 cursor-pointer">
						<input
							bind:group="{selectedAddress}"
							type="radio"
							value="{a._id}"
							name="group"
							class="mt-1.5 focus:outline-none focus:ring-0 focus:ring-offset-0" />
						<div class="w-full font-light  text-gray-800 cursor-pointer ms-2">
							<!-- on:change="{() => addressChanged(a._id)}" /> -->
							<h5 class="capitalize font-semibold tracking-wide md:text-lg">
								{a.firstName}
								{a.lastName}
							</h5>

							<div class="text-sm md:text-base flex items-start my-1 sm:w-2/3">
								<h5
									class="font-semibold tracking-wide me-2 w-16 sm:w-20 flex items-center justify-between">
									<span>Address</span> <span>:</span>
								</h5>

								<h6 class="flex flex-col">
									<span>{a.address},</span><span>{a.city}, {a.state}, {a.country}</span>
								</h6>
							</div>

							<div class="text-sm md:text-base  flex items-start my-1 sm:w-2/3">
								<h5
									class="font-semibold tracking-wide me-2 w-16 sm:w-20 flex items-center justify-between">
									<span>Pin</span> <span>:</span>
								</h5>

								<h6>{a.zip}</h6>
							</div>

							<div class="text-sm md:text-base  flex items-start my-1 sm:w-2/3">
								<h5
									class="font-semibold tracking-wide me-2 w-16 sm:w-20 flex items-center justify-between">
									<span>Phone</span> <span>:</span>
								</h5>

								<h6>{a.phone}</h6>
							</div>

							<div class="text-sm md:text-base  flex items-start my-1 sm:w-2/3">
								<h5
									class="font-semibold tracking-wide me-2 w-16 sm:w-20 flex items-center justify-between">
									<span>Email</span> <span>:</span>
								</h5>

								<h6>{a.email}</h6>
							</div>
						</div>
					</label>

					<div
						class="mx-8 sm:mx-10 mt-5 flex items-center text-sm sm:text-base space-x-5 sm:space-x-10">
						<button
							type="button"
							class="py-2 px-4 rounded-md shadow-md border border-primary-500 hover:bg-primary-500 hover:text-white text-primary-500 focus:outline-none w-full transition duration-300 font-semibold tracking-wide"
							on:click="{() => edit(a.id)}">
							EDIT
						</button>
						<button
							type="button"
							class="py-2 px-4 rounded-md hover:shadow-md border border-transparent hover:border-gray-500 bg-transparent hover:bg-gray-500 hover:text-white text-gray-500 focus:outline-none w-full transition duration-300 font-semibold tracking-wide"
							on:click="{() => remove(a._id)}">
							{#if iconloading}
								<div class="flex justify-center">
									<svg
										style="height: 20px; width: 20px"
										class="text-gray-500 -ms-1 animate-spin"
										xmlns="http://www.w3.org/2000/svg"
										fill="none"
										viewBox="0 0 24 24">
										<circle
											class="opacity-25"
											cx="12"
											cy="12"
											r="10"
											stroke="currentColor"
											stroke-width="4"></circle>
										<path
											class="opacity-75"
											fill="currentColor"
											d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"
										></path>
									</svg>
								</div>
							{:else}
								<span>REMOVE</span>
							{/if}
						</button>
					</div>
				</div>
			{/each}
			{#if addresses?.data?.length < 1}
				<div class="p-6 text-lg text-center text-gray-500">No Address found</div>
			{/if}
		</div>
	{/if}
</section>
