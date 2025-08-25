<script setup lang="ts">
import { createColumnHelper } from '@tanstack/vue-table'
import { ref, reactive, computed, h } from 'vue'
import DataTable from './DataTable.vue'
import Button from './ui/Button.vue'
import Input from './ui/Input.vue'
import Label from './ui/Label.vue'
import ConfirmActionDialog from './ConfirmActionDialog.vue'

// Types
interface Product {
  id: number;
  name: string;
  category: string;
  price: number;
  stockQuantity: number;
  status: "in_stock" | "out_of_stock" | "discontinued";
  createdAt?: string;
}

// Data
const products = ref<Product[]>([
  { id: 1, name: 'Product 1', category: 'Category 1', price: 100, stockQuantity: 50, status: 'in_stock', createdAt: '2024-01-15' },
  { id: 2, name: 'Product 2', category: 'Category 2', price: 200, stockQuantity: 30, status: 'out_of_stock', createdAt: '2024-01-14' },
  { id: 3, name: 'Product 3', category: 'Category 3', price: 300, stockQuantity: 20, status: 'discontinued', createdAt: '2024-01-13' },
  { id: 4, name: 'Product 4', category: 'Category 4', price: 400, stockQuantity: 5, status: 'in_stock', createdAt: '2024-01-12' },
])

const loading = ref(false)
const showProductForm = ref(false)
const showDeleteDialog = ref(false)
const editingProduct = ref<Product | null>(null)
const productToDelete = ref<Product | null>(null)

// Form data
const productForm = reactive({
  name: '',
  category: '',
  price: 0,
  stockQuantity: 0,
  status: 'in_stock' as Product['status'],
})

// Table setup
const columnHelper = createColumnHelper<Product>()

const formatPrice = (price: number) => {
    return new Intl.NumberFormat('en-US', {
        style: 'currency',
        currency: 'USD',
    }).format(price)
}

const columns = [
  columnHelper.accessor('name', {
    header: 'Name',
    cell: ({ getValue }) => getValue(),
  }),
  columnHelper.accessor('category', {
    header: 'Category',
    cell: ({ getValue }) => getValue(),
  }),
  columnHelper.accessor('price', {
    header: 'Price',
    cell: ({ getValue }) => formatPrice(getValue()),
  }),
  columnHelper.accessor('stockQuantity', {
    header: 'Stock Quantity',
    cell: ({ getValue }) => {
      const stockQuantity = getValue()

      return h('span', {
        class: `${stockQuantity < 10 ? 'text-red-600 font-bold' : 'text-gray-900'}`,
      }, stockQuantity)
    },
  }),
  columnHelper.accessor('status', {
    header: 'Status',
    cell: ({ getValue }) => {
      const status = getValue()
      const statusColors = {
        in_stock: 'bg-green-100 text-green-800',
        out_of_stock: 'bg-red-100 text-red-800',
        discontinued: 'bg-gray-100 text-gray-800',
      }
      return h('span', {
        class: `inline-flex px-2 py-1 text-xs font-semibold rounded-full ${statusColors[status]}`
      }, status.charAt(0).toUpperCase() + status.slice(1))
    },
  }),
  columnHelper.display({
    id: 'actions',
    header: 'Actions',
    cell: ({ row: { original } }) => {
      return h('div', { class: 'flex gap-2' }, [
        h(Button, {
          variant: 'outline',
          size: 'sm',
          onClick: () => editProduct(original)
        }, () => 'Edit'),
        h(Button, {
          variant: 'destructive',
          size: 'sm',
          onClick: () => confirmDelete(original)
        }, () => 'Delete'),
      ])
    },
  }),
]

// Computed
const isEditing = computed(() => editingProduct.value !== null)

// Methods
const resetForm = () => {
  productForm.name = ''
  productForm.category = ''
  productForm.price = 0
  productForm.stockQuantity = 0
  productForm.status = 'in_stock'
}

const openProductForm = () => {
  resetForm()
  editingProduct.value = null
  showProductForm.value = true
}

const editProduct = (product: Product) => {
  editingProduct.value = product
  productForm.name = product.name
  productForm.category = product.category
  productForm.price = product.price
  productForm.stockQuantity = product.stockQuantity
  productForm.status = product.status
  showProductForm.value = true
}

const saveProduct = () => {
  if (!productForm.name || !productForm.category) return

  loading.value = true

  // Simulate API call
  setTimeout(() => {

    
    if (isEditing.value && editingProduct.value) {
      // Update existing product

      const index = products.value.findIndex(u => u.id === editingProduct.value!.id)

      products.value = products.value.map((p) => {
        if (p.id === editingProduct.value!.id) {
          return {
            ...p,
            name: productForm.name,
            category: productForm.category,
            price: productForm.price,
            stockQuantity: productForm.stockQuantity,
            status: productForm.status,
          }
        }
        return p
      })
      
    } else {
      // Add new product
      const newProduct: Product = {
        id: Math.max(...products.value.map(u => u.id)) + 1,
        name: productForm.name,
        category: productForm.category,
        price: productForm.price,
        stockQuantity: productForm.stockQuantity,
        status: productForm.status,
      }
      products.value = [...products.value, newProduct]
    }

    loading.value = false
    showProductForm.value = false
    editingProduct.value = null
  }, 1000)
}

const confirmDelete = (product: Product) => {
  productToDelete.value = product
  showDeleteDialog.value = true
}

const deleteProduct = () => {
  if (productToDelete.value) {
    products.value = products.value.filter(u => u.id !== productToDelete.value!.id)
    productToDelete.value = null
    showDeleteDialog.value = false
  }
}

const cancelDelete = () => {
  productToDelete.value = null
  showDeleteDialog.value = false
}

const cancelForm = () => {
  showProductForm.value = false
  editingProduct.value = null
  resetForm()
}
</script>

<template>
  <div class="space-y-6">
    <!-- Header -->
    <div class="flex items-center justify-between">
      <div>
        <h1 class="text-3xl font-bold text-gray-900">Product Management</h1>
        <p class="text-gray-500">Manage products, categories, and inventory</p>
      </div>
      <Button @click="openProductForm" class="bg-blue-600 hover:bg-blue-700">
        Add New Product
      </Button>
    </div>

    <!-- User Table -->
    <div class="bg-white rounded-lg shadow">
      <DataTable :data="products" :columns="columns" :loading="false" />
    </div>

    <!-- User Form Modal -->
    <div v-if="showProductForm" class="fixed inset-0 z-50 flex items-center justify-center bg-black bg-opacity-50">
      <div class="bg-white rounded-lg p-6 w-full max-w-md mx-4">
        <h2 class="text-xl font-semibold mb-4">
          {{ isEditing ? 'Edit Product' : 'Add New Product' }}
        </h2>

        <form @submit.prevent="saveProduct" class="space-y-4">
          <div>
            <Label for="name">Name</Label>
            <Input id="name" v-model="productForm.name" placeholder="Enter product name" required class="mt-1" />
          </div>

          <div>
            <Label for="category">Category</Label>
            <Input id="category" v-model="productForm.category" type="string" placeholder="Enter category" required
              class="mt-1" />
          </div>

          <div>
            <Label for="price">Price</Label>
            <Input id="price" v-model.number="productForm.price" type="number" placeholder="Enter price" required
              class="mt-1" />
          </div>

          <div>
            <Label for="stockQuantity">Stock Quantity</Label>
            <Input id="stockQuantity" v-model.number="productForm.stockQuantity" type="number" placeholder="Enter stock quantity" required
              class="mt-1" />
          </div>

          <div>
            <Label for="status">Status</Label>
            <select id="status" v-model="productForm.status"
              class="mt-1 flex h-10 w-full rounded-md border border-input bg-background px-3 py-2 text-sm ring-offset-background focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2">
              <option value="in_stock">In Stock</option>
              <option value="out_of_stock">Out of Stock</option>
              <option value="discontinued">Discontinued</option>
            </select>
          </div>

          <div class="flex gap-3 pt-4">
            <Button type="submit" :disabled="loading" class="flex-1 bg-blue-600 hover:bg-blue-700">
              {{ loading ? 'Saving...' : (isEditing ? 'Update Product' : 'Create Product') }}
            </Button>
            <Button type="button" variant="outline" @click="cancelForm" class="flex-1">
              Cancel
            </Button>
          </div>
        </form>
      </div>
    </div>

    <!-- Delete Confirmation Dialog -->
    <ConfirmActionDialog v-model:open="showDeleteDialog" title="Delete Product"
      :description="`Are you sure you want to delete ${productToDelete?.name}? This action cannot be undone.`"
      action-variant="destructive" @continue-action="deleteProduct" @cancel-action="cancelDelete" />
  </div>
</template>
