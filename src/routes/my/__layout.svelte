<script context="module" lang="ts">
import type { Load } from '@sveltejs/kit'
export const load: Load = async ({ page: { path }, session: { user } }) => {
	if (!user) {
		return {
			redirect: `/login?ref=${path}`,
			status: 302,
		}
	}
	return {}
}
</script>

<script>
import SidebarDashboard from './_SidebarDashboard.svelte'
import SummaryDashboard from './_SummaryDashboard.svelte'
import OrdersDashboard from './_OrdersDashboard.svelte'
import SEO from '$lib/components/SEO/index.svelte'

const seoProps = {
	title: 'Dashboard',
	metadescription: 'Track your all process',
}
</script>

<SEO {...seoProps} />

<section class="w-full lg:h-screen md:pt-5 lg:pt-0">
	<div class="w-full p-2 sm:p-6 flex items-start ">
		<div>
			<SidebarDashboard />
		</div>

		<div class="w-full">
			<slot />
		</div>
	</div>

	<!-- <div class="lg:w-1/5">
		<OrdersDashboard />
	</div> -->
</section>
