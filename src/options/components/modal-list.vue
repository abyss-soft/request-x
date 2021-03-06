<template>
  <vl-modal :visible="!!store.editList" @close="onListCancel" transition="fade">
    <form class="w-1/2 mx-auto p-2 bg-white" v-if="store.editList" @submit.prevent="onListSave">
      <h3 class="font-bold mb-2" v-text="modalTitle" />
      <div class="nav-modal-group" v-if="!store.editList.isSubscribed">
        <div>Name:</div>
        <input v-model="store.editList.name" :placeholder="store.editList.defaultName">
      </div>
      <div class="nav-modal-group" v-if="store.editList.isSubscribed">
        <div>Subscribe URL:</div>
        <input
          :class="{ error: errors.subscribeUrl }"
          :value="store.editList.subscribeUrl"
          :readonly="store.editList.isEdit"
          @input="store.editList.isEdit || (store.editList.subscribeUrl = $event.target.value)"
        />
      </div>
      <div class="mt-1 text-right">
        <button class="mr-1" type="submit">OK</button>
        <button @click.prevent="onListCancel">Cancel</button>
      </div>
    </form>
  </vl-modal>
</template>

<script>
import {
  debounce, store, isValidURL, dump, pickData,
} from '../util';

export default {
  data() {
    return {
      store,
      errors: {},
    };
  },
  computed: {
    modalTitle() {
      if (!store.editList) return null;
      if (store.editList.editing) return 'Edit list';
      if (store.editList.isSubscribed) return 'Subscribe list';
      return 'Create list';
    },
  },
  watch: {
    'store.editList': {
      deep: true,
      handler() {
        this.updateErrors();
      },
    },
  },
  methods: {
    checkErrors() {
      const editList = this.store.editList || {};
      this.errors = {
        subscribeUrl: editList.isSubscribed && !isValidURL(editList.subscribeUrl),
      };
    },
    onListCancel() {
      store.editList = null;
    },
    onListSave() {
      this.checkErrors();
      if (Object.keys(this.errors).some(key => this.errors[key])) return;
      dump(pickData(this.store.editList, ['id', 'name', 'subscribeUrl']));
      this.onListCancel();
    },
  },
  created() {
    this.updateErrors = debounce(this.checkErrors, 200);
  },
};
</script>
