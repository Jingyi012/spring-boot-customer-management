<template>
  <div class="modal fade" tabindex="-1" ref="modal">
    <div class="modal-dialog">
      <div class="modal-content">
        <form @submit.prevent="handleSubmit">
          <div class="modal-header">
            <h5 class="modal-title">{{ isEdit ? 'Edit Customer' : 'Add Customer' }}</h5>
            <button type="button" class="btn-close" @click="close"></button>
          </div>
          <div class="modal-body">
            <div class="mb-3">
              <label class="form-label">First Name</label>
              <input v-model="form.firstName" type="text" class="form-control" required />
            </div>
            <div class="mb-3">
              <label class="form-label">Last Name</label>
              <input v-model="form.lastName" type="text" class="form-control" required />
            </div>
            <div class="mb-3">
              <label class="form-label">Email</label>
              <input v-model="form.email" type="email" class="form-control" required />
            </div>
            <div class="mb-3">
              <label class="form-label">Phone</label>
              <input v-model="form.phone" type="text" class="form-control" required />
            </div>
          </div>
          <div class="modal-footer">
            <button type="button" class="btn btn-secondary" @click="close">Cancel</button>
            <button type="submit" class="btn btn-primary">{{ isEdit ? 'Update' : 'Add' }}</button>
          </div>
        </form>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'CustomerFormModal',
  props: {
    customer: { type: Object, default: null }
  },
  data() {
    return {
      form: {
        firstName: '',
        lastName: '',
        email: '',
        phone: ''
      },
      modalInstance: null
    }
  },
  computed: {
    isEdit() {
      return !!this.customer
    }
  },
  watch: {
    customer: {
      immediate: true,
      handler(newVal) {
        if (newVal) {
          this.form = { ...newVal }
        } else {
          this.form = { firstName: '', lastName: '', email: '', phone: '' }
        }
      }
    }
  },
  methods: {
    show() {
      this.modalInstance = new window.bootstrap.Modal(this.$refs.modal)
      this.modalInstance.show()
    },
    close() {
      if (this.modalInstance) this.modalInstance.hide()
      this.$emit('close')
    },
    handleSubmit() {
      this.$emit('save', { ...this.form })
      this.close()
    }
  }
}
</script>