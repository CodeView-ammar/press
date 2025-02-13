<template>
	<div>
		<PageHeader title="Benches" subtitle="Private benches you own">
			<template v-slot:actions>
				<Button
					appearance="primary"
					iconLeft="plus"
					class="ml-2"
					@click="showBillingDialog"
				>
					New
				</Button>
			</template>
		</PageHeader>

		<SectionHeader :heading="getBenchFilterHeading()">
			<template #actions>
				<Dropdown :options="benchFilterOptions()">
					<template v-slot="{ open }">
						<Button
							:class="[
								'rounded-md px-3 py-1 text-base font-medium',
								open ? 'bg-gray-200' : 'bg-gray-100'
							]"
							icon-left="chevron-down"
							>{{ benchFilter.replace('tag:', '') }}</Button
						>
					</template>
				</Dropdown>
			</template>
		</SectionHeader>

		<div class="mt-3">
			<LoadingText v-if="$resources.allBenches.loading" />
			<BenchList v-else :benches="benches" />
		</div>

		<Dialog
			:options="{ title: 'Add card to create new benches' }"
			v-model="showAddCardDialog"
		>
			<template v-slot:body-content>
				<StripeCard
					class="mb-1"
					v-if="showAddCardDialog"
					@complete="
						showAddCardDialog = false;
						$resources.paymentMethods.reload();
					"
				/>
			</template>
		</Dialog>
	</div>
</template>

<script>
import BenchList from './BenchList.vue';
import { defineAsyncComponent } from 'vue';

export default {
	name: 'BenchesScreen',
	data() {
		return {
			showAddCardDialog: false,
			benchFilter: 'All'
		};
	},
	pageMeta() {
		return {
			title: 'Benches - Frappe Cloud'
		};
	},
	components: {
		BenchList,
		StripeCard: defineAsyncComponent(() =>
			import('@/components/StripeCard.vue')
		)
	},
	resources: {
		paymentMethods: 'press.api.billing.get_payment_methods',
		allBenches() {
			return {
				method: 'press.api.bench.all',
				params: { bench_filter: this.benchFilter },
				auto: true
			};
		},
		benchTags: 'press.api.bench.bench_tags'
	},
	computed: {
		benches() {
			if (!this.$resources.allBenches.data) {
				return [];
			}

			return this.$resources.allBenches.data;
		}
	},
	methods: {
		benchFilterOptions() {
			const options = [
				{
					group: 'Status',
					items: [
						{
							label: 'All',
							handler: () => (this.benchFilter = 'All')
						},
						{
							label: 'Active',
							handler: () => (this.benchFilter = 'Active')
						},
						{
							label: 'Awaiting Deploy',
							handler: () => (this.benchFilter = 'Awaiting Deploy')
						}
					]
				}
			];

			if (!this.$resources.benchTags?.data?.length) return options;

			return [
				...options,
				{
					group: 'Tags',
					items: this.$resources.benchTags.data.map(tag => ({
						label: tag,
						handler: () => (this.benchFilter = `tag:${tag}`)
					}))
				}
			];
		},
		getBenchFilterHeading() {
			if (this.benchFilter === 'Awaiting Deploy')
				return 'Benches Awaiting Deploy';
			else if (this.benchFilter.startsWith('tag:'))
				return `Benches with tag ${this.benchFilter.slice(4)}`;
			return `${this.benchFilter || 'All'} Benches`;
		},
		showBillingDialog() {
			if (!this.$account.hasBillingInfo) {
				this.showAddCardDialog = true;
			} else {
				this.$router.replace('/benches/new');
			}
		}
	}
};
</script>
