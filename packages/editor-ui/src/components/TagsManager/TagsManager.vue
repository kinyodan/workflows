<template>
	<Modal
		:title="$locale.baseText('tagsManager.manageTags')"
		:name="TAGS_MANAGER_MODAL_KEY"
		:eventBus="modalBus"
		@enter="onEnter"
		minWidth="620px"
		minHeight="420px"
	>
		<template v-slot:content>
			<el-row>
				<TagsView
					v-if="hasTags || isCreating"
					:isLoading="isLoading"
					:tags="tags"

					@create="onCreate"
					@update="onUpdate"
					@delete="onDelete"
					@disableCreate="onDisableCreate"
				/>
				<NoTagsView
					@enableCreate="onEnableCreate"
					v-else />
			</el-row>
		</template>
		<template v-slot:footer="{ close }">
			<n8n-button :label="$locale.baseText('tagsManager.done')" @click="close" float="right" />
		</template>
	</Modal>
</template>

<script lang="ts">
import Vue from "vue";
import mixins from "vue-typed-mixins";
import { mapGetters } from "vuex";

import { ITag } from "@/Interface";

import { showMessage } from "@/components/mixins/showMessage";
import TagsView from "@/components/TagsManager/TagsView/TagsView.vue";
import NoTagsView from "@/components/TagsManager/NoTagsView.vue";
import Modal from "@/components/Modal.vue";
import { TAGS_MANAGER_MODAL_KEY } from '../../constants';

export default mixins(showMessage).extend({
	name: "TagsManager",
	created() {
		this.$store.dispatch("tags/fetchAll", {force: true, withUsageCount: true});
	},
	data() {
		const tagIds = (this.$store.getters['tags/allTags'] as ITag[])
			.map((tag) => tag.id);

		return {
			tagIds,
			isCreating: false,
			modalBus: new Vue(),
			TAGS_MANAGER_MODAL_KEY,
		};
	},
	components: {
		TagsView,
		NoTagsView,
		Modal,
	},
	computed: {
		...mapGetters("tags", ["isLoading"]),
		tags(): ITag[] {
			return this.$data.tagIds.map((tagId: string) => this.$store.getters['tags/getTagById'](tagId))
				.filter(Boolean); // if tag is deleted from store
		},
		hasTags(): boolean {
			return this.tags.length > 0;
		},
	},
	methods: {
		onEnableCreate() {
			this.$data.isCreating = true;
		},

		onDisableCreate() {
			this.$data.isCreating = false;
		},

		async onCreate(name: string, cb: (tag: ITag | null, error?: Error) => void) {
			try {
				if (!name) {
					throw new Error(
						this.$locale.baseText('tagsManager.tagNameCannotBeEmpty'),
					);
				}

				const newTag = await this.$store.dispatch("tags/create", name);
				this.$data.tagIds = [newTag.id].concat(this.$data.tagIds);
				cb(newTag);
			} catch (error) {
				const escapedName = escape(name);
				this.$showError(
					error,
					this.$locale.baseText('tagsManager.showError.onCreate.title'),
					this.$locale.baseText(
						'tagsManager.showError.onCreate.message',
						{ interpolate: { escapedName } },
					) + ':',
				);
				cb(null, error);
			}
		},

		async onUpdate(id: string, name: string, cb: (tag: boolean, error?: Error) => void) {
			const tag = this.$store.getters['tags/getTagById'](id);
			const oldName = tag.name;

			try {
				if (!name) {
					throw new Error(
						this.$locale.baseText('tagsManager.tagNameCannotBeEmpty'),
					);
				}

				if (name === oldName) {
					cb(true);
					return;
				}

				const updatedTag = await this.$store.dispatch("tags/rename", { id, name });
				cb(!!updatedTag);

				const escapedName = escape(name);
				const escapedOldName = escape(oldName);

				this.$showMessage({
					title: this.$locale.baseText('tagsManager.showMessage.onUpdate.title'),
					message: this.$locale.baseText(
						'tagsManager.showMessage.onUpdate.message',
						{ interpolate: { escapedName, escapedOldName } },
					),
					type: "success",
				});
			} catch (error) {
				const escapedName = escape(oldName);
				this.$showError(
					error,
					this.$locale.baseText('tagsManager.showError.onUpdate.title'),
					this.$locale.baseText(
						'tagsManager.showError.onUpdate.message',
						{ interpolate: { escapedName } },
					) + ':',
				);
				cb(false, error);
			}
		},

		async onDelete(id: string, cb: (deleted: boolean, error?: Error) => void) {
			const tag = this.$store.getters['tags/getTagById'](id);
			const name = tag.name;

			try {
				const deleted = await this.$store.dispatch("tags/delete", id);
				if (!deleted) {
					throw new Error(
						this.$locale.baseText('tagsManager.couldNotDeleteTag'),
					);
				}

				this.$data.tagIds = this.$data.tagIds.filter((tagId: string) => tagId !== id);

				cb(deleted);

				const escapedName = escape(name);
				this.$showMessage({
					title: this.$locale.baseText('tagsManager.showMessage.onDelete.title'),
					message: this.$locale.baseText(
						'tagsManager.showMessage.onDelete.message',
						{ interpolate: { escapedName } },
					),
					type: "success",
				});
			} catch (error) {
				const escapedName = escape(name);
				this.$showError(
					error,
					this.$locale.baseText('tagsManager.showError.onDelete.title'),
					this.$locale.baseText(
						'tagsManager.showError.onDelete.message',
						{ interpolate: { escapedName } },
					) + ':',
				);
				cb(false, error);
			}
		},

		onEnter() {
			if (this.isLoading) {
				return;
			}
			else if (!this.hasTags) {
				this.onEnableCreate();
			}
			else {
				this.modalBus.$emit('close');
			}
		},
	},
});
</script>

