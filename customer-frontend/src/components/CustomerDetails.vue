<template>
  <div class="container py-4">
    <!-- Loading State -->
    <div v-if="loading" class="d-flex justify-content-center py-5">
      <div class="spinner-border text-primary" role="status">
        <span class="visually-hidden">Loading...</span>
      </div>
    </div>

    <!-- Customer Details/Edit -->
    <div v-else-if="currentCustomer" class="card shadow-sm">
      <div class="card-header bg-white border-bottom-0">
        <div class="d-flex flex-wrap align-items-center justify-content-between">
          <div class="d-flex align-items-center mb-3 mb-md-0">
            <div class="bg-primary text-white rounded-circle d-flex align-items-center justify-content-center"
              style="width: 60px; height: 60px; font-size: 1.5rem; font-weight: bold;">
              {{ currentCustomer.firstName.charAt(0) }}{{ currentCustomer.lastName.charAt(0) }}
            </div>
            <div class="ms-3">
              <h2 class="mb-0">{{ currentCustomer.firstName }} {{ currentCustomer.lastName }}</h2>
              <small class="text-muted">Customer ID: {{ currentCustomer.id }}</small>
            </div>
          </div>
          <div class="d-flex flex-wrap gap-2 align-items-center">
            <!-- Toggle Edit Button -->
            <button
                @click="toggleEditMode"
                :class="['btn', editMode ? 'btn-outline-secondary' : 'btn-outline-primary']"
                data-bs-toggle="tooltip"
                :title="editMode ? 'Cancel Editing' : 'Edit Customer'"
            >
                <i :class="['bi', editMode ? 'bi-x-circle' : 'bi-pencil']"></i>
                {{ editMode ? 'Cancel' : 'Edit' }}
            </button>

            <!-- Save Button (only in edit mode) -->
            <button
                v-if="editMode"
                @click="saveCustomer"
                class="btn btn-success"
                data-bs-toggle="tooltip"
                title="Save Changes"
            >
                <i class="bi bi-check-circle"></i> Save
            </button>

            <!-- Delete Button (only when NOT in edit mode) -->
            <button
                v-if="!editMode"
                @click="showDeleteModal"
                class="btn btn-outline-danger"
                data-bs-toggle="tooltip"
                title="Delete this customer"
            >
                <i class="bi bi-trash"></i> Delete
            </button>
          </div>
        </div>
      </div>

      <div class="card-body">
        <div class="mb-4">
          <h5 class="text-primary mb-3"><i class="bi bi-person me-2"></i>Personal Information</h5>
          <div class="row">
            <div class="col-md-6 mb-3">
              <label class="form-label text-muted mb-1">First Name</label>
              <p v-if="!editMode" class="mb-0">{{ currentCustomer.firstName }}</p>
              <input v-else v-model="editableCustomer.firstName" class="form-control" />
            </div>
            <div class="col-md-6 mb-3">
              <label class="form-label text-muted mb-1">Last Name</label>
              <p v-if="!editMode" class="mb-0">{{ currentCustomer.lastName }}</p>
              <input v-else v-model="editableCustomer.lastName" class="form-control" />
            </div>
          </div>
        </div>

        <div class="mb-4">
          <h5 class="text-primary mb-3"><i class="bi bi-envelope me-2"></i>Contact Information</h5>
          <div class="row">
            <div class="col-md-6 mb-3">
              <label class="form-label text-muted mb-1">Email</label>
              <p v-if="!editMode" class="mb-0">
                <a :href="`mailto:${currentCustomer.email}`" class="text-decoration-none">
                  {{ currentCustomer.email }}
                </a>
              </p>
              <input v-else v-model="editableCustomer.email" class="form-control" type="email" />
            </div>
            <div class="col-md-6 mb-3">
              <label class="form-label text-muted mb-1">Phone</label>
              <p v-if="!editMode" class="mb-0">
                <a :href="`tel:${currentCustomer.phone}`" class="text-decoration-none">
                  {{ currentCustomer.phone }}
                </a>
              </p>
              <input v-else v-model="editableCustomer.phone" class="form-control" />
            </div>
          </div>
        </div>
      </div>

      <div class="card-footer bg-white border-top-0 text-center">
        <button @click="$router.go(-1)" class="btn btn-outline-secondary">
          <i class="bi bi-arrow-left"></i> Back to List
        </button>
      </div>
    </div>

    <!-- Not Found State -->
    <div v-else class="text-center py-5">
      <div class="text-danger mb-3" style="font-size: 3rem;">
        <i class="bi bi-exclamation-triangle"></i>
      </div>
      <h3 class="mb-3">Customer Not Found</h3>
      <p class="text-muted mb-4">The requested customer could not be found.</p>
      <button @click="$router.push({ name: 'customers' })" class="btn btn-primary">
        Return to Customer List
      </button>
    </div>

    <!-- Success Message -->
    <transition name="fade">
      <div v-if="message" class="alert alert-success alert-dismissible fade show position-fixed bottom-0 end-0 m-3" style="z-index: 1000;">
        {{ message }}
        <button type="button" class="btn-close" @click="message = ''"></button>
      </div>
    </transition>

    <!-- Delete Confirmation Modal (Bootstrap style, same as CustomerList) -->
    <div class="modal fade" id="deleteModal" tabindex="-1" aria-labelledby="deleteModalLabel" aria-hidden="true" ref="deleteModal">
      <div class="modal-dialog modal-dialog-top">
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title" id="deleteModalLabel">Confirm Deletion</h5>
            <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close" @click="hideDeleteModal"></button>
          </div>
          <div class="modal-body">
            <p>Are you sure you want to delete <strong>{{ currentCustomer.firstName }} {{ currentCustomer.lastName }}</strong>?</p>
            <p class="text-danger"><strong>This action cannot be undone.</strong></p>
          </div>
          <div class="modal-footer">
            <button type="button" class="btn btn-secondary" data-bs-dismiss="modal" @click="hideDeleteModal">
              Cancel
            </button>
            <button type="button" class="btn btn-danger" @click="confirmDelete">
              Delete Customer
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import CustomerDataService from '../services/CustomerDataService'

export default {
  name: 'CustomerDetails',
  data() {
    return {
      currentCustomer: null,
      editableCustomer: null,
      editMode: false,
      loading: true,
      message: '',
      deleteModalInstance: null
    }
  },
  methods: {
    getCustomer(id) {
      this.loading = true
      CustomerDataService.get(id)
        .then(response => {
          this.currentCustomer = response.data
        })
        .catch(e => {
          console.error(e)
          this.message = 'Failed to load customer details'
        })
        .finally(() => {
          this.loading = false
        })
    },
    toggleEditMode() {
      this.editMode = !this.editMode
      if (this.editMode) {
        this.editableCustomer = { ...this.currentCustomer }
      }
    },
    saveCustomer() {
      CustomerDataService.update(this.currentCustomer.id, this.editableCustomer)
        .then(() => {
          this.currentCustomer = { ...this.editableCustomer }
          this.message = 'Customer updated successfully'
          this.editMode = false
        })
        .catch(e => {
          console.error(e)
          this.message = 'Failed to update customer'
        })
    },
    showDeleteModal() {
      const modal = new window.bootstrap.Modal(this.$refs.deleteModal)
      modal.show()
      this.deleteModalInstance = modal
    },
    hideDeleteModal() {
      if (this.deleteModalInstance) this.deleteModalInstance.hide()
    },
    confirmDelete() {
      this.hideDeleteModal()
      CustomerDataService.delete(this.currentCustomer.id)
        .then(() => {
          this.message = 'Customer deleted successfully'
          setTimeout(() => {
            this.$router.push({ name: 'customers' })
          }, 1500)
        })
        .catch(e => {
          console.error(e)
          this.message = 'Failed to delete customer'
        })
    }
  },
  mounted() {
    this.getCustomer(this.$route.params.id)
  }
}
</script>

<style scoped>
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s;
}
.fade-enter,
.fade-leave-to {
  opacity: 0;
}
.modal.fade.show {
  backdrop-filter: blur(2px);
}
</style>
