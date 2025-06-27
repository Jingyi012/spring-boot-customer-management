<template>
    <div>
        <transition name="fade">
            <div
                v-if="message"
                class="alert alert-dismissible fade show position-fixed top-0 start-50 translate-middle-x mt-3"
                :class="{'alert-success': messageType === 'success', 'alert-danger': messageType === 'error'}"
                role="alert"
                style="z-index: 2000; min-width: 300px; max-width: 90vw;"
            >
                {{ message }}
                <button type="button" class="btn-close" @click="message = ''" aria-label="Close"></button>
            </div>
        </transition>
        <div class="d-flex justify-content-between align-items-center mb-3">
            <h2>Customer List</h2>
            <div>
                <button class="btn btn-outline-primary me-2" @click="exportExcel">
                    <i class="bi bi-file-earmark-excel"></i> Export Excel
                </button>
                <button class="btn btn-success" @click="openAddModal">
                    <i class="bi bi-plus-lg"></i> New
                </button>
            </div>
        </div>

        <div v-if="loading" class="text-center my-5">
            <div class="spinner-border text-primary" role="status">
                <span class="visually-hidden">Loading...</span>
            </div>
        </div>
        <table v-if="!loading" class="table table-striped table-hover align-middle">
            <thead class="table-dark">
                <tr>
                    <th scope="col">First Name</th>
                    <th scope="col">Last Name</th>
                    <th scope="col">Email</th>
                    <th scope="col">Phone</th>
                    <th scope="col" class="text-center">Actions</th>
                </tr>
            </thead>
            <tbody>
                <tr v-for="(customer, index) in customers" :key="index">
                    <td>{{ customer.firstName }}</td>
                    <td>{{ customer.lastName }}</td>
                    <td>{{ customer.email }}</td>
                    <td>{{ customer.phone }}</td>
                    <td class="text-center">
                        <button @click="viewDetails(customer)" class="btn btn-sm btn-outline-info me-2">
                            <i class="bi bi-eye"></i> View
                        </button>
                        <button @click="openEditModal(customer)" class="btn btn-sm btn-outline-primary me-2">
                            <i class="bi bi-pencil"></i> Edit
                        </button>
                        <button @click="openDeleteModal(customer)" class="btn btn-sm btn-outline-danger">
                            <i class="bi bi-trash"></i> Delete
                        </button>
                    </td>
                </tr>
            </tbody>
        </table>
        <div class="modal fade" id="deleteModal" tabindex="-1" aria-labelledby="deleteModalLabel" aria-hidden="true" ref="deleteModal">
        <div class="modal-dialog">
          <div class="modal-content">
            <div class="modal-header">
              <h5 class="modal-title" id="deleteModalLabel">Confirm Delete</h5>
              <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
            </div>
            <div class="modal-body">
              Are you sure you want to delete 
              <strong>
                  {{ customerToDelete && customerToDelete.firstName }} 
                  {{ customerToDelete && customerToDelete.lastName }}
              </strong>?
              </div>

            <div class="modal-footer">
              <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">
                  Cancel
              </button>
              <button type="button" class="btn btn-danger" @click="deleteCustomer">
                  Delete
              </button>
            </div>
          </div>
        </div>
      </div>
      <CustomerFormModal
        ref="addModal"
        v-if="showAdd"
        @save="addCustomer"
        @close="showAdd = false"
      />
      <CustomerFormModal
        ref="editModal"
        v-if="showEdit"
        :customer="customerToEdit"
        @save="updateCustomer"
        @close="showEdit = false"
      />
    </div>
</template>

<script>
import CustomerDataService from '../services/CustomerDataService'
import CustomerFormModal from './CustomerFormModal.vue'

export default {
  name: 'customers',
  components: { CustomerFormModal },
  data() {
    return {
      customers: [],
      message: '',
      messageType: 'success',
      loading: false,
      customerToDelete: null,
      customerToEdit: null,
      showAdd: false,
      showEdit: false
    }
  },
  methods: {
    retrieveCustomers() {
      this.loading = true
      CustomerDataService.getAll()
        .then(response => {
          this.customers = response.data
        })
        .catch(() => {
          this.showMessage('Failed to load customers.', 'error')
        })
        .finally(() => {
          this.loading = false
        })
    },
    openAddModal() {
      this.showAdd = true
      this.$nextTick(() => this.$refs.addModal.show())
    },
    addCustomer(newCustomer) {
      CustomerDataService.create(newCustomer)
        .then(() => {
          this.showMessage('Customer added successfully!', 'success')
          this.retrieveCustomers()
        })
        .catch(() => {
          this.showMessage('Failed to add customer.', 'error')
        })
    },
    openEditModal(customer) {
      this.customerToEdit = { ...customer }
      this.showEdit = true
      this.$nextTick(() => this.$refs.editModal.show())
    },
    updateCustomer(updatedCustomer) {
      CustomerDataService.update(updatedCustomer.id, updatedCustomer)
        .then(() => {
          this.showMessage('Customer updated successfully!', 'success')
          this.retrieveCustomers()
        })
        .catch(() => {
          this.showMessage('Failed to update customer.', 'error')
        })
    },
    openDeleteModal(customer) {
      this.customerToDelete = customer
      // Show Bootstrap modal
      const modal = new window.bootstrap.Modal(this.$refs.deleteModal)
      modal.show()
      this.deleteModalInstance = modal
    },
    deleteCustomer() {
      if (!this.customerToDelete) return
      CustomerDataService.delete(this.customerToDelete.id)
        .then(() => {
          this.showMessage('Customer deleted successfully!', 'success')
          this.retrieveCustomers()
        })
        .catch(() => {
          this.showMessage('Failed to delete customer.', 'error')
        })
        .finally(() => {
          this.deleteModalInstance.hide()
          this.customerToDelete = null
        })
    },
    showMessage(msg, type) {
      this.message = msg
      this.messageType = type
      setTimeout(() => {
        this.message = ''
      }, 2000)
    },
    exportExcel() {
      fetch('http://localhost:8080/api/customers/export', {
        method: 'GET'
      })
        .then(response => {
          if (!response.ok) throw new Error('Network response was not ok')
          return response.blob()
        })
        .then(blob => {
          const url = window.URL.createObjectURL(blob)
          const a = document.createElement('a')
          a.href = url
          a.download = 'customers.xlsx'
          document.body.appendChild(a)
          a.click()
          a.remove()
          window.URL.revokeObjectURL(url)
        })
        .catch(() => {
          this.showMessage('Failed to export Excel.', 'error')
        })
    },
    viewDetails(customer) {
      this.$router.push(`/customers/${customer.id}`)
    },
  },
  mounted() {
    this.retrieveCustomers()
  }
}
</script>
<style>
.fade-enter-active, .fade-leave-active {
  transition: opacity 0.3s;
}

.fade-enter, .fade-leave-to {
  opacity: 0;
}
</style>