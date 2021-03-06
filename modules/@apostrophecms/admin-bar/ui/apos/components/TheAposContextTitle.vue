<template>
  <transition-group
    tag="div"
    class="apos-admin-bar__control-set apos-admin-bar__control-set--title"
    name="flip"
  >
    <span
      v-show="true"
      class="apos-admin-bar__title"
      :key="'title'"
    >
      <AposIndicator
        icon="information-outline-icon"
        fill-color="var(--a-primary)"
        :tooltip="docTooltip"
        class="apos-admin-bar__title__indicator"
      />
      <span class="apos-admin-bar__title__document-title">
        {{ context.title }}
      </span>
      <span class="apos-admin-bar__title__separator">
        —
      </span>
      <AposContextMenu
        v-if="!isUnpublished"
        class="apos-admin-bar__title__document"
        :button="draftButton"
        :menu="draftMenu"
        :disabled="hasCustomUi || isUnpublished"
        :tooltip="isUnpublished ? 'The current document is unpublished' : null"
        @item-clicked="switchDraftMode"
        menu-offset="13, 10"
        menu-placement="bottom-end"
      />
      <AposLabel
        v-else
        label="Draft" :modifiers="['apos-is-warning', 'apos-is-filled']"
        tooltip="This document has not been published"
      />
    </span>
  </transition-group>
</template>
<script>
import dayjs from 'dayjs';

export default {
  name: 'TheAposContextTitle',
  props: {
    draftMode: {
      type: String,
      required: true
    },
    hasCustomUi: Boolean,
    context: {
      type: Object,
      default() {
        return {};
      }
    }
  },
  emits: [ 'switchDraftMode' ],
  computed: {
    updatedBy() {
      let editorLabel = 'ApostropheCMS ■●▲';
      if (this.context.updatedBy) {
        const editor = this.context.updatedBy;
        editorLabel = '';
        editorLabel += editor.title ? `${editor.title} ` : '';
        editorLabel += editor.username ? `(${editor.username})` : '';
      }
      return editorLabel;
    },
    draftButton() {
      return {
        label: (this.draftMode === 'draft') ? 'Draft' : 'Published',
        icon: 'chevron-down-icon',
        modifiers: [ 'icon-right', 'no-motion' ],
        type: 'quiet'
      };
    },
    isUnpublished() {
      return !this.context.lastPublishedAt;
    },
    docTooltip() {
      return `Last saved on ${dayjs(this.context.updatedAt).format('ddd MMMM D [at] H:mma')} <br /> by ${this.updatedBy}`;
    },
    draftMenu() {
      return [
        {
          label: 'Draft',
          name: 'draft',
          action: 'draft',
          modifiers: (this.draftMode === 'draft') ? [ 'disabled', 'selected' ] : null
        },
        {
          label: 'Published',
          name: 'published',
          action: 'published',
          modifiers: (this.draftMode === 'published') ? [ 'disabled', 'selected' ] : null
        }
      ];
    }
  },
  methods: {
    switchDraftMode(mode) {
      this.$emit('switchDraftMode', mode);
    }
  }
};
</script>
<style lang="scss" scoped>
.apos-admin-bar__control-set--title {
  justify-content: center;
  align-items: center;
}
.apos-admin-bar__title {
  display: inline-flex;
  align-items: center;
  white-space: nowrap;

  &__document-title,
  &__separator {
    display: inline-flex;
    color: var(--a-text-primary);
  }

  &__document-title {
    margin-top: 1px;
  }

  &__separator {
    align-items: center;
    padding: 0 7px;
    margin-top: 1px;
  }

  &__document {
    margin-top: 3.5px;
  }
}

.apos-admin-bar__title__indicator {
  margin-right: 5px;
  color: var(--a-text-primary);
}
</style>
