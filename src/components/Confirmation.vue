<template>
  <div>
    <b-alert variant="success"
      show=""
      dismissible
      v-model="showAlert"
      @dismissed="dismissAlert"
    >
      {{ message }}
    </b-alert>
  </div>
</template>

<script>
export default {
  props: {
    message: {
      type: String,
    },
    showAlert: {
      type: Boolean,
    },
  },
  methods: {
    dismissAlert() {
      this.$emit('dismiss-alert');
    },
  },
};
</script>

<style scoped>

</style>
