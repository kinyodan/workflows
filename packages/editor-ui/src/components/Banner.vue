<template>
	<el-tag
		:type="theme"
		size="medium"
		:disable-transitions="true"
		:class="$style.container"
	>
		<font-awesome-icon
				:icon="theme === 'success' ? 'check-circle' : 'exclamation-triangle'"
				:class="theme === 'success' ? $style.icon : $style.dangerIcon"
		/>
		<div
			:class="$style.banner"
		>
			<div :class="$style.content">
				<div>
					<span
						:class="theme === 'success' ? $style.message : $style.dangerMessage"
					>
						{{ message }}&nbsp;
					</span>
					<a v-if="details && !expanded" :class="$style.expandButton" @click="expand">More details</a>
				</div>
			</div>

			<n8n-button
				v-if="buttonLabel"
				:label="buttonLoading && buttonLoadingLabel ? buttonLoadingLabel : buttonLabel"
				:title="buttonTitle"
				:theme="theme"
				:loading="buttonLoading"
				size="small"
				type="outline"
				:transparentBackground="true"
				@click.stop="onClick"
			/>
		</div>

		<div v-if="expanded" :class="$style.details">
			{{details}}
		</div>
	</el-tag>
</template>

<script lang="ts">
import Vue from 'vue';

export default Vue.extend({
	name: 'Banner',
	data() {
		return {
			expanded: false,
		};
	},
	props: {
		theme: {
			type: String,
			validator: (value: string): boolean =>
				['success', 'danger'].indexOf(value) !== -1,
		},
		message: {
			type: String,
		},
		buttonLabel: {
			type: String,
		},
		buttonLoadingLabel: {
			type: String,
		},
		buttonTitle: {
			type: String,
		},
		details: {
			type: String,
		},
		buttonLoading: {
			type: Boolean,
			default: false,
		},
	},
	methods: {
		expand() {
			this.expanded = true;
		},
		onClick() {
			this.expanded = false;
			this.$emit('click');
		},
	},
});
</script>

<style module lang="scss">
.icon {
	position: absolute;
	left: 14px;
	top: 18px;
}

.dangerIcon {
	composes: icon;
	color: var(--color-danger);
}

.container {
	width: 100%;
	position: relative;
	padding-left: 40px;
	border: none;
}

.message {
	white-space: normal;
	line-height: var(--font-line-height-regular);
	overflow: hidden;
	word-break: break-word;
}

.dangerMessage {
	composes: message;
	color: var(--color-danger);
}

.banner {
	display: flex;
	align-items: center;
}

.content {
	flex-grow: 1;
	min-height: 26px;
	display: flex;
	align-items: center;
}

.expandButton {
	font-weight: var(--font-weight-bold);
}

.details {
	composes: message;
	margin-top: var(--spacing-3xs);
	color: var(--color-text-base);
	font-size: var(--font-size-2xs);
}

</style>
