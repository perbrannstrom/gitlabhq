<script>
  import Flash from '../../../flash';
  import editForm from './edit_form.vue';
  import issuableMixin from '../../../vue_shared/mixins/issuable';
  import Icon from '../../../vue_shared/components/icon.vue';

  export default {
    components: {
      editForm,
      Icon,
    },
    mixins: [
      issuableMixin,
    ],

    props: {
      isLocked: {
        required: true,
        type: Boolean,
      },

      isEditable: {
        required: true,
        type: Boolean,
      },

      mediator: {
        required: true,
        type: Object,
        validator(mediatorObject) {
          return mediatorObject.service && mediatorObject.service.update && mediatorObject.store;
        },
      },
    },

    computed: {
      lockIcon() {
        return this.isLocked ? 'lock' : 'lock-open';
      },

      isLockDialogOpen() {
        return this.mediator.store.isLockDialogOpen;
      },
    },

    methods: {
      toggleForm() {
        this.mediator.store.isLockDialogOpen = !this.mediator.store.isLockDialogOpen;
      },

      updateLockedAttribute(locked) {
        this.mediator.service.update(this.issuableType, {
          discussion_locked: locked,
        })
        .then(() => location.reload())
        .catch(() => Flash(this.__(`Something went wrong trying to
  change the locked state of this ${this.issuableDisplayName}`)));
      },
    },
  };
</script>

<template>
  <div class="block issuable-sidebar-item lock">
    <div class="sidebar-collapsed-icon">
      <icon
        :name="lockIcon"
        :size="16"
        aria-hidden="true"
        class="sidebar-item-icon is-active"
      />
    </div>

    <div class="title hide-collapsed">
      Lock {{ issuableDisplayName }}
      <button
        v-if="isEditable"
        class="pull-right lock-edit btn btn-blank"
        type="button"
        @click.prevent="toggleForm"
      >
        {{ __('Edit') }}
      </button>
    </div>

    <div class="value sidebar-item-value hide-collapsed">
      <edit-form
        v-if="isLockDialogOpen"
        :toggle-form="toggleForm"
        :is-locked="isLocked"
        :update-locked-attribute="updateLockedAttribute"
        :issuable-type="issuableType"
      />

      <div
        v-if="isLocked"
        class="value sidebar-item-value"
      >
        <icon
          name="lock"
          :size="16"
          aria-hidden="true"
          class="sidebar-item-icon inline is-active"
        />
        {{ __('Locked') }}
      </div>

      <div
        v-else
        class="no-value sidebar-item-value hide-collapsed"
      >
        <icon
          name="lock-open"
          :size="16"
          aria-hidden="true"
          class="sidebar-item-icon inline"
        />
        {{ __('Unlocked') }}
      </div>
    </div>
  </div>
</template>
