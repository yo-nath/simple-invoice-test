<template>
	<v-dialog v-model="modal" max-width="500px" :persistent="true">
		<v-card>
			<v-card-title>
				<span class="headline">{{ title }}</span>
			</v-card-title>

			<v-card-text>
				<v-container grid-list-md>
					<slot name="fields"></slot>
				</v-container>
			</v-card-text>

			<v-card-actions>
				<v-spacer></v-spacer>
				<slot name="actions"></slot>
			</v-card-actions>
		</v-card>
	</v-dialog>
</template>

<script>
export default {
	props: ['title', 'show'],
	computed: {
		modal() {
			return this.show
		}
	}
}
</script>

