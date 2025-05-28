<script setup lang="ts">
import { ref, computed, reactive } from 'vue'
import { 
  TrendingUp, 
  Package, 
  Plus, 
  DollarSign, 
  ShoppingCart, 
  AlertTriangle, 
  Search, 
  Edit, 
  Upload, 
  CheckCircle, 
  ImageIcon,
  Users
} from 'lucide-vue-next'

// Active tab state
const activeTab = ref('revenue')

// Revenue data
const revenueData = ref([
  { period: 'Jan', orders: 145, revenue: 12500, inventory: 850 },
  { period: 'Feb', orders: 178, revenue: 15200, inventory: 920 },
  { period: 'Mar', orders: 203, revenue: 18900, inventory: 780 },
  { period: 'Apr', orders: 189, revenue: 16800, inventory: 650 },
  { period: 'May', orders: 234, revenue: 21400, inventory: 890 },
  { period: 'Jun', orders: 267, revenue: 24600, inventory: 720 }
])

const categoryData = ref([
  { name: 'Electronics', value: 35, revenue: 45000, color: '#3b82f6' },
  { name: 'Clothing', value: 25, revenue: 32000, color: '#10b981' },
  { name: 'Home & Garden', value: 20, revenue: 28000, color: '#f59e0b' },
  { name: 'Sports', value: 12, revenue: 18000, color: '#ef4444' },
  { name: 'Books', value: 8, revenue: 12000, color: '#8b5cf6' }
])

// Filters for revenue
const timeFilter = ref('monthly')
const categoryFilter = ref('all')

// Computed revenue metrics
const totalRevenue = computed(() => revenueData.value.reduce((sum, item) => sum + item.revenue, 0))
const totalOrders = computed(() => revenueData.value.reduce((sum, item) => sum + item.orders, 0))
const avgOrderValue = computed(() => totalRevenue.value / totalOrders.value)

// Product interface
interface Product {
  id: string
  name: string
  category: string
  price: number
  stock: number
  lowStockThreshold: number
  image: string
  description: string
  lastUpdated: string
}

// Products data
const products = ref<Product[]>([
  {
    id: '1',
    name: 'Wireless Headphones',
    category: 'Electronics',
    price: 99.99,
    stock: 15,
    lowStockThreshold: 10,
    image: 'src/assets/headphonesreal.jpg',
    description: 'High-quality wireless headphones with noise cancellation',
    lastUpdated: '2024-01-15'
  },
  {
    id: '2',
    name: 'Cotton T-Shirt',
    category: 'Clothing',
    price: 24.99,
    stock: 5,
    lowStockThreshold: 20,
    image: 'src/assets/tshirt.jpg',
    description: 'Comfortable 100% cotton t-shirt',
    lastUpdated: '2024-01-14'
  },
  {
    id: '3',
    name: 'Garden Hose',
    category: 'Home & Garden',
    price: 39.99,
    stock: 25,
    lowStockThreshold: 15,
    image: 'src/assets/gardenhose.jpg',
    description: '50ft expandable garden hose',
    lastUpdated: '2024-01-13'
  },
  {
    id: '4',
    name: 'Running Shoes',
    category: 'Sports',
    price: 89.99,
    stock: 3,
    lowStockThreshold: 10,
    image: 'src/assets/shoesreal.jpg',
    description: 'Lightweight running shoes for athletes',
    lastUpdated: '2024-01-12'
  },
  {
    id: '5',
    name: 'Programming Book',
    category: 'Books',
    price: 49.99,
    stock: 12,
    lowStockThreshold: 5,
    image: 'src/assets/book.jpg',
    description: 'Complete guide to modern web development',
    lastUpdated: '2024-01-11'
  }
])

// Inventory management
const searchTerm = ref('')
const inventoryCategoryFilter = ref('all')
const sortBy = ref('name')
const editingStock = ref<string | null>(null)
const newStockValue = ref('')

// Product registration form
const productForm = reactive({
  name: '',
  description: '',
  category: '',
  price: '',
  stock: '',
  lowStockThreshold: '',
  image: null as File | null
})

const isSubmitting = ref(false)
const submitSuccess = ref(false)
const imagePreview = ref<string | null>(null)

// Computed properties
const filteredProducts = computed(() => {
  return products.value
    .filter(product => 
      product.name.toLowerCase().includes(searchTerm.value.toLowerCase()) &&
      (inventoryCategoryFilter.value === 'all' || product.category === inventoryCategoryFilter.value)
    )
    .sort((a, b) => {
      switch (sortBy.value) {
        case 'name':
          return a.name.localeCompare(b.name)
        case 'stock':
          return a.stock - b.stock
        case 'price':
          return a.price - b.price
        default:
          return 0
      }
    })
})

const lowStockProducts = computed(() => 
  products.value.filter(product => product.stock <= product.lowStockThreshold)
)

const totalInventoryValue = computed(() => 
  products.value.reduce((sum, product) => sum + (product.price * product.stock), 0)
)

const isFormValid = computed(() => 
  productForm.name && productForm.description && productForm.category && productForm.price && productForm.stock
)

// Chart helpers
const maxRevenueValue = computed(() => Math.max(...revenueData.value.map(d => d.revenue)))
const maxOrderValue = computed(() => Math.max(...revenueData.value.map(d => d.orders)))

const getBarHeight = (value: number, max: number) => (value / max) * 100
const getStockStatus = (product: Product) => {
  if (product.stock === 0) return { label: 'Out of Stock', class: 'status-out-of-stock' }
  if (product.stock <= product.lowStockThreshold) return { label: 'Low Stock', class: 'status-low-stock' }
  return { label: 'In Stock', class: 'status-in-stock' }
}

// Methods
const updateStock = (productId: string, newStock: number) => {
  const productIndex = products.value.findIndex(p => p.id === productId)
  if (productIndex !== -1) {
    products.value[productIndex].stock = newStock
    products.value[productIndex].lastUpdated = new Date().toISOString().split('T')[0]
  }
  editingStock.value = null
  newStockValue.value = ''
}

const handleImageChange = (event: Event) => {
  const target = event.target as HTMLInputElement
  const file = target.files?.[0]
  if (file) {
    productForm.image = file
    
    const reader = new FileReader()
    reader.onload = (e) => {
      imagePreview.value = e.target?.result as string
    }
    reader.readAsDataURL(file)
  }
}

const handleSubmit = async () => {
  isSubmitting.value = true
  
  // Simulate API call
  await new Promise(resolve => setTimeout(resolve, 2000))
  
  // Add product to inventory
  const newProduct: Product = {
    id: Date.now().toString(),
    name: productForm.name,
    description: productForm.description,
    category: productForm.category,
    price: parseFloat(productForm.price),
    stock: parseInt(productForm.stock),
    lowStockThreshold: parseInt(productForm.lowStockThreshold) || 10,
    image: imagePreview.value || '/placeholder.svg?height=50&width=50',
    lastUpdated: new Date().toISOString().split('T')[0]
  }
  
  products.value.push(newProduct)
  
  isSubmitting.value = false
  submitSuccess.value = true
  
  // Reset form
  setTimeout(() => {
    Object.assign(productForm, {
      name: '',
      description: '',
      category: '',
      price: '',
      stock: '',
      lowStockThreshold: '',
      image: null
    })
    imagePreview.value = null
    submitSuccess.value = false
  }, 3000)
}

const clearForm = () => {
  Object.assign(productForm, {
    name: '',
    description: '',
    category: '',
    price: '',
    stock: '',
    lowStockThreshold: '',
    image: null
  })
  imagePreview.value = null
}

const renderStars = (rating: number) => {
  return Array.from({ length: 5 }, (_, i) => i < rating)
}
</script>

<template>
  <div class="dashboard-container">
    <!-- Header -->
    <div class="header">
      <div class="header-content">
        <h1 class="header-title">E-commerce Admin Dashboard</h1>
        <p class="header-subtitle">Manage your store, track revenue, and monitor inventory</p>
      </div>
    </div>

    <div class="main-content">
      <!-- Tab Navigation -->
      <div class="tab-navigation">
        <nav class="tab-nav">
          <button
            @click="activeTab = 'revenue'"
            :class="['tab-button', { 'tab-active': activeTab === 'revenue' }]"
          >
            <TrendingUp class="tab-icon" />
            Revenue Analysis
          </button>
          <button
            @click="activeTab = 'inventory'"
            :class="['tab-button', { 'tab-active': activeTab === 'inventory' }]"
          >
            <Package class="tab-icon" />
            Inventory Management
          </button>
          <button
            @click="activeTab = 'products'"
            :class="['tab-button', { 'tab-active': activeTab === 'products' }]"
          >
            <Plus class="tab-icon" />
            Add Product
          </button>
        </nav>
      </div>

      <!-- Revenue Analysis Tab -->
      <div v-if="activeTab === 'revenue'" class="tab-content">
        <!-- Filters -->
        <div class="filters">
          <select v-model="timeFilter" class="filter-select">
            <option value="daily">Daily</option>
            <option value="weekly">Weekly</option>
            <option value="monthly">Monthly</option>
            <option value="annually">Annually</option>
          </select>
          
          <select v-model="categoryFilter" class="filter-select">
            <option value="all">All Categories</option>
            <option value="electronics">Electronics</option>
            <option value="clothing">Clothing</option>
            <option value="home">Home & Garden</option>
            <option value="sports">Sports</option>
            <option value="books">Books</option>
          </select>
        </div>

        <!-- Key Metrics -->
        <div class="metrics-grid">
          <div class="metric-card">
            <div class="metric-content">
              <div class="metric-info">
                <p class="metric-label">Total Revenue</p>
                <p class="metric-value">${{ totalRevenue.toLocaleString() }}</p>
                <p class="metric-change">+12% from last period</p>
              </div>
              <DollarSign class="metric-icon metric-icon-blue" />
            </div>
          </div>

          <div class="metric-card">
            <div class="metric-content">
              <div class="metric-info">
                <p class="metric-label">Total Orders</p>
                <p class="metric-value">{{ totalOrders.toLocaleString() }}</p>
                <p class="metric-change">+8% from last period</p>
              </div>
              <ShoppingCart class="metric-icon metric-icon-green" />
            </div>
          </div>

          <div class="metric-card">
            <div class="metric-content">
              <div class="metric-info">
                <p class="metric-label">Avg Order Value</p>
                <p class="metric-value">${{ avgOrderValue.toFixed(2) }}</p>
                <p class="metric-change">+3% from last period</p>
              </div>
              <TrendingUp class="metric-icon metric-icon-purple" />
            </div>
          </div>

          <div class="metric-card">
            <div class="metric-content">
              <div class="metric-info">
                <p class="metric-label">Conversion Rate</p>
                <p class="metric-value">3.2%</p>
                <p class="metric-change">+0.5% from last period</p>
              </div>
              <Users class="metric-icon metric-icon-orange" />
            </div>
          </div>
        </div>

        <!-- Charts -->
        <div class="charts-grid">
          <!-- Revenue Chart -->
          <div class="chart-card">
            <h3 class="chart-title">Revenue & Orders Trend</h3>
            <div class="chart-container">
              <svg viewBox="0 0 100 100" class="chart-svg">
                <!-- Grid -->
                <defs>
                  <pattern id="grid" width="10" height="10" patternUnits="userSpaceOnUse">
                    <path d="M 10 0 L 0 0 0 10" fill="none" stroke="#f3f4f6" stroke-width="0.5"/>
                  </pattern>
                </defs>
                <rect width="100" height="100" fill="url(#grid)" />
                
                <!-- Revenue Bars -->
                <g v-for="(item, index) in revenueData" :key="`revenue-${index}`">
                  <rect
                    :x="index * 15 + 2"
                    :y="100 - getBarHeight(item.revenue, maxRevenueValue)"
                    width="6"
                    :height="getBarHeight(item.revenue, maxRevenueValue)"
                    fill="#3b82f6"
                    rx="1"
                  />
                  <!-- Order Bars -->
                  <rect
                    :x="index * 15 + 9"
                    :y="100 - getBarHeight(item.orders, maxOrderValue)"
                    width="6"
                    :height="getBarHeight(item.orders, maxOrderValue)"
                    fill="#10b981"
                    rx="1"
                  />
                  <text
                    :x="index * 15 + 7.5"
                    y="98"
                    text-anchor="middle"
                    class="chart-label"
                  >
                    {{ item.period }}
                  </text>
                </g>
              </svg>
            </div>
            <div class="chart-legend">
              <div class="legend-item">
                <div class="legend-color legend-blue"></div>
                <span class="legend-text">Revenue</span>
              </div>
              <div class="legend-item">
                <div class="legend-color legend-green"></div>
                <span class="legend-text">Orders</span>
              </div>
            </div>
          </div>

          <!-- Category Distribution -->
          <div class="chart-card">
            <h3 class="chart-title">Revenue by Category</h3>
            <div class="category-list">
              <div v-for="category in categoryData" :key="category.name" class="category-item">
                <div class="category-info">
                  <div class="category-color" :style="{ backgroundColor: category.color }"></div>
                  <span class="category-name">{{ category.name }}</span>
                </div>
                <div class="category-stats">
                  <div class="category-revenue">${{ category.revenue.toLocaleString() }}</div>
                  <div class="category-percentage">{{ category.value }}%</div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- Inventory Management Tab -->
      <div v-if="activeTab === 'inventory'" class="tab-content">
        <!-- Low Stock Alert -->
        <div v-if="lowStockProducts.length > 0" class="alert-box">
          <div class="alert-content">
            <AlertTriangle class="alert-icon" />
            <div>
              <h4 class="alert-title">Low Stock Alert</h4>
              <p class="alert-text">
                {{ lowStockProducts.length }} product(s) need restocking: 
                {{ lowStockProducts.map(p => p.name).join(', ') }}
              </p>
            </div>
          </div>
        </div>

        <!-- Filters and Search -->
        <div class="inventory-controls">
          <div class="search-filters">
            <div class="search-container">
              <Search class="search-icon" />
              <input
                v-model="searchTerm"
                type="text"
                placeholder="Search products..."
                class="search-input"
              />
            </div>

            <select v-model="inventoryCategoryFilter" class="filter-select">
              <option value="all">All Categories</option>
              <option value="Electronics">Electronics</option>
              <option value="Clothing">Clothing</option>
              <option value="Home & Garden">Home & Garden</option>
              <option value="Sports">Sports</option>
              <option value="Books">Books</option>
            </select>

            <select v-model="sortBy" class="filter-select">
              <option value="name">Sort by Name</option>
              <option value="stock">Sort by Stock</option>
              <option value="price">Sort by Price</option>
            </select>
          </div>

          <!-- Products Table -->
          <div class="table-container">
            <table class="products-table">
              <thead>
                <tr class="table-header">
                  <th class="table-th">Product</th>
                  <th class="table-th">Category</th>
                  <th class="table-th">Price</th>
                  <th class="table-th">Stock</th>
                  <th class="table-th">Status</th>
                  <th class="table-th">Last Updated</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="product in filteredProducts" :key="product.id" class="table-row">
                  <td class="table-td">
                    <div class="product-info">
                      <img :src="product.image" :alt="product.name" class="product-image" />
                      <div>
                        <div class="product-name">{{ product.name }}</div>
                        <div class="product-description">{{ product.description }}</div>
                      </div>
                    </div>
                  </td>
                  <td class="table-td">{{ product.category }}</td>
                  <td class="table-td">${{ product.price }}</td>
                  <td class="table-td">
                    <div v-if="editingStock === product.id" class="stock-edit">
                      <input
                        v-model="newStockValue"
                        type="number"
                        min="0"
                        class="stock-input"
                      />
                      <button
                        @click="updateStock(product.id, parseInt(newStockValue))"
                        class="stock-save-btn"
                        :disabled="!newStockValue || parseInt(newStockValue) < 0"
                      >
                        Save
                      </button>
                      <button
                        @click="editingStock = null; newStockValue = ''"
                        class="stock-cancel-btn"
                      >
                        Cancel
                      </button>
                    </div>
                    <div v-else class="stock-display">
                      <span>{{ product.stock }}</span>
                      <button
                        @click="editingStock = product.id; newStockValue = product.stock.toString()"
                        class="stock-edit-btn"
                      >
                        <Edit class="edit-icon" />
                      </button>
                    </div>
                  </td>
                  <td class="table-td">
                    <span :class="['status-badge', getStockStatus(product).class]">
                      {{ getStockStatus(product).label }}
                    </span>
                  </td>
                  <td class="table-td">{{ product.lastUpdated }}</td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>

        <!-- Inventory Summary -->
        <div class="summary-grid">
          <div class="summary-card">
            <div class="summary-content">
              <div class="summary-info">
                <p class="summary-label">Total Products</p>
                <p class="summary-value">{{ products.length }}</p>
                <p class="summary-note">Across all categories</p>
              </div>
              <Package class="summary-icon summary-icon-blue" />
            </div>
          </div>

          <div class="summary-card">
            <div class="summary-content">
              <div class="summary-info">
                <p class="summary-label">Low Stock Items</p>
                <p class="summary-value summary-value-orange">{{ lowStockProducts.length }}</p>
                <p class="summary-note">Need immediate attention</p>
              </div>
              <AlertTriangle class="summary-icon summary-icon-orange" />
            </div>
          </div>

          <div class="summary-card">
            <div class="summary-content">
              <div class="summary-info">
                <p class="summary-label">Total Inventory Value</p>
                <p class="summary-value">${{ totalInventoryValue.toLocaleString() }}</p>
                <p class="summary-note">Current stock value</p>
              </div>
              <DollarSign class="summary-icon summary-icon-green" />
            </div>
          </div>
        </div>
      </div>

      <!-- Product Registration Tab -->
      <div v-if="activeTab === 'products'" class="product-form-container">
        <!-- Success Message -->
        <div v-if="submitSuccess" class="success-message">
          <div class="success-content">
            <CheckCircle class="success-icon" />
            <div class="success-text">
              <h4 class="success-title">Success!</h4>
              <p class="success-description">Product successfully added to inventory! The dashboard will be updated shortly.</p>
            </div>
          </div>
        </div>

        <!-- Product Form -->
        <div class="form-card">
          <h2 class="form-title">Add New Product</h2>
          <p class="form-description">Register a new product in your inventory system. All fields marked with * are required.</p>

          <form @submit.prevent="handleSubmit" class="product-form">
            <!-- Image Upload -->
            <div class="form-group">
              <label class="form-label">Product Image</label>
              <div class="image-upload-container">
                <div class="image-upload-area">
                  <div class="upload-zone">
                    <input
                      type="file"
                      accept="image/*"
                      @change="handleImageChange"
                      class="file-input"
                      id="image-upload"
                    />
                    <label for="image-upload" class="upload-label">
                      <ImageIcon class="upload-icon" />
                      <p class="upload-text">Click to upload product image</p>
                      <p class="upload-hint">PNG, JPG, GIF up to 10MB</p>
                    </label>
                  </div>
                </div>
                <div v-if="imagePreview" class="image-preview">
                  <img :src="imagePreview" alt="Preview" class="preview-image" />
                </div>
              </div>
            </div>

            <!-- Product Name -->
            <div class="form-group">
              <label class="form-label">Product Name *</label>
              <input
                v-model="productForm.name"
                type="text"
                required
                placeholder="Enter product name"
                class="form-input"
              />
            </div>

            <!-- Description -->
            <div class="form-group">
              <label class="form-label">Description *</label>
              <textarea
                v-model="productForm.description"
                required
                rows="3"
                placeholder="Describe your product..."
                class="form-textarea"
              ></textarea>
            </div>

            <!-- Category and Price -->
            <div class="form-row">
              <div class="form-group">
                <label class="form-label">Category *</label>
                <select
                  v-model="productForm.category"
                  required
                  class="form-select"
                >
                  <option value="">Select category</option>
                  <option value="Electronics">Electronics</option>
                  <option value="Clothing">Clothing</option>
                  <option value="Home & Garden">Home & Garden</option>
                  <option value="Sports">Sports</option>
                  <option value="Books">Books</option>
                  <option value="Beauty">Beauty</option>
                  <option value="Toys">Toys</option>
                </select>
              </div>

              <div class="form-group">
                <label class="form-label">Price ($) *</label>
                <div class="input-with-icon">
                  <DollarSign class="input-icon" />
                  <input
                    v-model="productForm.price"
                    type="number"
                    step="0.01"
                    min="0"
                    required
                    placeholder="0.00"
                    class="form-input form-input-with-icon"
                  />
                </div>
              </div>
            </div>

            <!-- Stock Information -->
            <div class="form-row">
              <div class="form-group">
                <label class="form-label">Initial Stock Level *</label>
                <div class="input-with-icon">
                  <Package class="input-icon" />
                  <input
                    v-model="productForm.stock"
                    type="number"
                    min="0"
                    required
                    placeholder="0"
                    class="form-input form-input-with-icon"
                  />
                </div>
              </div>

              <div class="form-group">
                <label class="form-label">Low Stock Alert Threshold</label>
                <input
                  v-model="productForm.lowStockThreshold"
                  type="number"
                  min="0"
                  placeholder="10"
                  class="form-input"
                />
                <p class="form-hint">You'll be alerted when stock falls below this level</p>
              </div>
            </div>

            <!-- Submit Buttons -->
            <div class="form-actions">
              <button
                type="button"
                @click="clearForm"
                class="btn-secondary"
              >
                Clear Form
              </button>
              <button
                type="submit"
                :disabled="!isFormValid || isSubmitting"
                class="btn-primary"
              >
                <div v-if="isSubmitting" class="btn-loading">
                  <div class="spinner"></div>
                  <span>Adding...</span>
                </div>
                <div v-else class="btn-content">
                  <Upload class="btn-icon" />
                  <span>Add Product</span>
                </div>
              </button>
            </div>
          </form>
        </div>

        <!-- Product Preview -->
        <div v-if="productForm.name || productForm.description" class="preview-card">
          <h3 class="preview-title">Product Preview</h3>
          <p class="preview-description">How your product will appear in the inventory</p>
          
          <div class="preview-content">
            <div class="preview-image-container">
              <img v-if="imagePreview" :src="imagePreview" alt="Preview" class="preview-product-image" />
              <Package v-else class="preview-placeholder-icon" />
            </div>
            <div class="preview-details">
              <h4 class="preview-product-name">{{ productForm.name || 'Product Name' }}</h4>
              <p class="preview-product-description">{{ productForm.description || 'Product description' }}</p>
              <div class="preview-product-meta">
                <span class="preview-price">${{ productForm.price || '0.00' }}</span>
                <span class="preview-stock">Stock: {{ productForm.stock || '0' }}</span>
                <span v-if="productForm.category" class="preview-category">
                  {{ productForm.category }}
                </span>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
/* Base Styles */
.dashboard-container {
  min-height: 100vh;
  background-color: #031201;
}

/* Header */
.header {
background: rgba(41, 60, 46, 1.5);
backdrop-filter: blur(10px);
-webkit-backdrop-filter: blur(15px); 
box-shadow: 0 4px 30px rgba(0, 0, 0, 0.1); 
border: 1px solid rgba(255, 255, 255, 0.2); 

}

.header-content {
  max-width: 1280px;
  margin: 0 auto;
  padding: 1rem 1.5rem;
}

.header-title {
  font-size: 2.1rem;
  font-weight: 700;
  color: #d8d8d8;
  margin: 0;
}

.header-subtitle {
  color: #a6aab1;
  margin: 0.25rem 0 0 0;
}

/* Main Content */
.main-content {
  max-width: 1280px;
  margin: 0 auto;
  padding: 1.5rem;
}

/* Tab Navigation */
.tab-navigation {
  border-bottom: 1px solid #f4f5f8;
  margin-bottom: 1.5rem;
}

.tab-nav {
  display: flex;
  gap: 2rem;
  margin-bottom: -1px;
}

.tab-button {
  display: flex;
  align-items: center;
  padding: 0.5rem 0.25rem;
  border-bottom: 2px solid transparent;
  font-weight: 500;
  font-size: 0.875rem;
  color: #cecfcf;
  background: none;
  border-top: none;
  border-left: none;
  border-right: none;
  cursor: pointer;
  transition: color 0.2s ease-in-out, border-color 0.2s ease-in-out;
}

.tab-button:hover {
  color: #374151;
  border-bottom-color: #d1d5db;
}

.tab-button.tab-active {
  border-bottom-color: #032002;
  color: #0d5d0a;
}

.tab-icon {
  width: 1rem;
  height: 1rem;
  margin-right: 0.5rem;
}

/* Tab Content */
.tab-content {
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
}

/* Filters */
.filters {
  display: flex;
  flex-wrap: wrap;
  gap: 1rem;
}

.filter-select {
padding: 0.5rem 0.75rem;
background: rgba(41, 60, 46, 1.5);
backdrop-filter: blur(10px);
-webkit-backdrop-filter: blur(15px); 
box-shadow: 0 4px 30px rgba(0, 0, 0, 0.1); 
border: 1px solid rgba(255, 255, 255, 0.2); 
font-size: 0.875rem;
border-radius: 0.5rem; /* Rounded corners */
color: #ffffff;  
}

.filter-select:focus {
  outline: none;
  border-color: #b2b2b2;
}

/* Metrics Grid */
.metrics-grid {
  display: grid;
  gap: 2rem;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
}

.metric-card {
  background: rgba(41, 60, 46, 1.5);
backdrop-filter: blur(10px);
-webkit-backdrop-filter: blur(15px); 
box-shadow: 0 4px 30px rgba(0, 0, 0, 0.1); 
border: 1px solid rgba(255, 255, 255, 0.2); 
  border-radius: 0.5rem;
  box-shadow: 0 1px 3px 0 rgba(0, 0, 0, 0.1);
  padding: 1.5rem;
}

.metric-content {
  display: flex;
  align-items: center;
  justify-content: space-between;
  color: #ffffff; 
}

.metric-label {
  font-size: 1.8rem;
  font-weight: 500;
  color: #ffffff; 
  margin: 0;
}

.metric-value {
  font-size: 2rem;
  font-weight: 700;
  color: #0b651d; 
  margin: 0;
}

.metric-change {
  font-size: 1rem;
  color: #ffffff; 
  margin: 0.25rem 0 0 0;
}

.metric-icon {
  width: 2rem;
  height: 2rem;
}

.metric-icon-blue { color: #3b82f6; }
.metric-icon-green { color: #10b981; }
.metric-icon-purple { color: #8b5cf6; }
.metric-icon-orange { color: #f59e0b; }
/* Charts */
.charts-grid {
  display: grid;
  gap: 1.5rem;
  grid-template-columns: repeat(auto-fit, minmax(400px, 1fr));
}

.chart-card {
  background: rgba(41, 60, 46, 1.5);
  backdrop-filter: blur(12px);
  -webkit-backdrop-filter: blur(12px);
  border-radius: 0.75rem;
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.2);
  padding: 1.5rem;
  border: 2px solid rgba(255, 255, 255, 0.1);
}

.chart-title {
  font-size: 2rem;
  font-weight: 600;
  color: #e5e7e6; /* Light greenish text for dark bg */
  margin: 0 0 1rem 0;
}

.chart-container {
  height: 20rem;
}

.chart-svg {
  width: 100%;
  height: 100%;
}

.chart-label {
  font-size: 0.4rem;
  fill: #f3f3f3; 
}

.chart-legend {
  display: flex;
  justify-content: center;
  gap: 2rem;
  margin-top: 1rem;
  font-size: 1rem;
}

.legend-item {
  display: flex;
  align-items: center;
}

.legend-color {
  width: 0.75rem;
  height: 0.75rem;
  font-size: 2rem;
  border-radius: 0.125rem;
  margin-right: 0.5rem;
}

.legend-blue {
  background-color: #d6ddd8; 
}

.legend-green {
  background-color: #22c55e; 
}

.legend-text {
  font-size: 1.5rem;
  color: #bbf7d0; /* soft mint text */
}


/* Category List */
.category-list {
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
}

.category-item {
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.category-info {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  color: #ffffff; 
}

.category-color {
  width: 1rem;
  height: 1rem;
  border-radius: 0.125rem;
}

.category-name {
  font-size: 0.875rem;
  font-weight: 500;
  font-size: 1.5rem;
  color: #ffffff; 
}

.category-stats {
  text-align: right;
}

.category-revenue {
  font-size: 1.5rem;
  font-weight: 600;
}

.category-percentage {
  font-size: 1rem;
  color: #d8dbe3;
}

/* Alert Box */
.alert-box {
  background-color: #570505;
  border: 1px solid #2d0303;
  border-radius: 0.5rem;
  padding: 1rem;
}

.alert-content {
  display: flex;
  align-items: flex-start;
}

.alert-icon {
  width: 1.25rem;
  height: 1.25rem;
  color: #cbcbcb;
  margin-right: 0.5rem;
  flex-shrink: 0;
}

.alert-title {
  font-weight: 600;
  color: #d5d5d5;
  margin: 0;
}

.alert-text {
  font-size: 0.875rem;
  color: #dbdbdb;
  margin: 0;
}

/* Inventory Controls */
.inventory-controls {
  background: rgba(41, 60, 46, 1.5);
  backdrop-filter: blur(12px);
  -webkit-backdrop-filter: blur(12px);
  border-radius: 0.75rem;
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.2);
  border-radius: 0.5rem;
  box-shadow: 0 1px 3px 0 rgba(0, 0, 0, 0.1);
  padding: 1.5rem;
}

.search-filters {
  display: flex;
  flex-wrap: wrap;
  gap: 1rem;
  margin-bottom: 1.5rem;
}

.search-container {
  position: relative;
  flex: 2;
  min-width: 120px;
}

.search-icon {
  position: absolute;
  left: 0.75rem;
  top: 0.625rem;
  width: 1rem;
  height: 1rem;
  color: #9ca3af;
}

.search-input {
  width: 50%;
  padding: 0.5rem 1rem 0.5rem 2.5rem;
  border: 1px solid #d1d5db;
  border-radius: 0.375rem;
  font-size: 0.875rem;
}

.search-input:focus {
  outline: none;
  border-color: #dae0e9;
}

/* Table */
.table-container {
  overflow-x: auto;
}

.products-table {
  width: 100%;
  border-collapse: collapse;
}

.table-header {
  border-bottom: 1px solid #e5e7eb;
}

.table-th {
  text-align: left;
  padding: 0.75rem 1rem;
  font-weight: 600;
  color: #dcdee1;
  font-size: 0.875rem;
}

.table-row {
  border-bottom: 1px solid #f3f4f6;
  transition: background-color 0.2s ease-in-out;
}

.table-row:hover {
  background-color: #011708;
}

.table-td {
  padding: 1rem;
  color: #e0e5ee;
}

.product-info {
  display: flex;
  align-items: center;
  gap: 0.75rem;
}

.product-image {
  width: 3rem;
  height: 3rem;
  border-radius: 0.5rem;
  object-fit: cover;
}

.product-name {
  font-weight: 700;
  color: #e4e4e4;
}

.product-description {
  font-size: 0.875rem;
  color: #b8bbc1;
}

/* Stock Management */
.stock-edit {
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.stock-input {
  width: 5rem;
  padding: 0.25rem 0.5rem;
  border: 1px solid #d1d5db;
  border-radius: 0.25rem;
  font-size: 0.875rem;
}

.stock-input:focus {
  outline: none;
  border-color: #3b82f6;
}

.stock-save-btn {
  padding: 0.25rem 0.5rem;
  background-color: #3b82f6;
  color: white;
  border: none;
  border-radius: 0.25rem;
  font-size: 0.875rem;
  cursor: pointer;
}

.stock-save-btn:hover {
  background-color: #2563eb;
}

.stock-save-btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.stock-cancel-btn {
  padding: 0.25rem 0.5rem;
  background-color: #d1d5db;
  color: #374151;
  border: none;
  border-radius: 0.25rem;
  font-size: 0.875rem;
  cursor: pointer;
}

.stock-cancel-btn:hover {
  background-color: #9ca3af;
}

.stock-display {
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.stock-edit-btn {
  padding: 0.25rem;
  color: #9ca3af;
  background: none;
  border: none;
  cursor: pointer;
}

.stock-edit-btn:hover {
  color: #6b7280;
}

.edit-icon {
  width: 1rem;
  height: 1rem;
}

/* Status Badges */
.status-badge {
  padding: 0.25rem 0.5rem;
  border-radius: 9999px;
  font-size: 0.75rem;
  font-weight: 500;
}

.status-out-of-stock {
  background-color: #fee2e2;
  color: #991b1b;
}

.status-low-stock {
  background-color: #fef3c7;
  color: #92400e;
}

.status-in-stock {
  background-color: #d1fae5;
  color: #065f46;
}

/* Summary Grid */
.summary-grid {
  display: grid;
  gap: 1rem;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
}

.summary-card {
  background: rgba(41, 60, 46, 1.5);
  backdrop-filter: blur(12px);
  -webkit-backdrop-filter: blur(12px);
  border-radius: 0.75rem;
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.2);
  border-radius: 0.5rem;
  box-shadow: 0 1px 3px 0 rgba(0, 0, 0, 0.1);
  padding: 1.5rem;
}

.summary-content {
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.summary-label {
  font-size: 0.875rem;
  font-weight: 1000;
  color: #e9ebee;
  margin: 0;
}

.summary-value {
  font-size: 1.5rem;
  font-weight: 700;
  color: #111827;
  margin: 0;
}

.summary-value-orange {
  color: #530303;
}

.summary-note {
  font-size: 0.75rem;
  color: #c6c8cb;
  margin: 0.25rem 0 0 0;
}

.summary-icon {
  width: 2rem;
  height: 2rem;
}

.summary-icon-blue { color: #3b82f6; }
.summary-icon-orange { color: #f59e0b; }
.summary-icon-green { color: #10b981; }

/* Product Form */
.product-form-container {
  background: rgba(140, 149, 142, 0.3); /* Corrected transparency */
  backdrop-filter: blur(10px);
  -webkit-backdrop-filter: blur(15px); 
  box-shadow: 0 4px 30px rgba(0, 0, 0, 0.1); 
  border: 1px solid rgba(255, 255, 255, 0.2);
  max-width: 48rem;
  margin: 0 auto;
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
}

.success-message {
  background-color: #d1fae5;
  border: 1px solid #03281c;
  border-radius: 0.5rem;
  padding: 1rem;
}

.success-content {
  display: flex;
  align-items: flex-start;
}

.success-icon {
  width: 1.25rem;
  height: 1.25rem;
  color: #059669;
  margin-right: 0.5rem;
  flex-shrink: 0;
}

.success-title {
  font-weight: 600;
  color: #065f46;
  margin: 0;
}

.success-description {
  font-size: 0.875rem;
  color: #047857;
  margin: 0;
}

.form-card {
  background: rgba(41, 60, 46, 1.5);/* Corrected transparency */
  backdrop-filter: blur(10px);
  -webkit-backdrop-filter: blur(15px); 
  box-shadow: 0 4px 30px rgba(0, 0, 0, 0.1); 
  border: 1px solid rgba(255, 255, 255, 0.2); 
  border-radius: 0.5rem;
  box-shadow: 0 1px 3px 0 rgba(0, 0, 0, 0.1);
  padding: 1.5rem;
}

.form-title {
  font-size: 1.25rem;
  font-weight: 600;
  color: #e9e9e9;
  margin: 0 0 1rem 0;
}

.form-description {
  color: #eceef4;
  margin: 0 0 1.5rem 0;
}

.product-form {
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
}

.form-group {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.form-label {
  display: block;
  font-size: 0.875rem;
  font-weight: 500;
  color: #e4e6ea;
}

.form-input {
  width: 50%;
  padding: 0.5rem 0.75rem;
  border: 1px solid #d1d5db;
  border-radius: 0.375rem;
  font-size: 0.875rem;
}

.form-input:focus {
  outline: none;
  border-color: #03230d;
}

.form-textarea {
  width: 70%;
  padding: 0.5rem 0.75rem;
  border: 1px solid #d1d5db;
  border-radius: 0.375rem;
  font-size: 0.875rem;
  resize: vertical;
}

.form-textarea:focus {
  outline: none;
  border-color: #032805;
}

.form-select {
  width: 100%;
  padding: 0.5rem 0.75rem;
  border: 1px solid #d1d5db;
  border-radius: 0.375rem;
  background-color: white;
  font-size: 0.875rem;
}

.form-select:focus {
  outline: none;
  border-color: #3b82f6;
}

.form-row {
  display: grid;
  grid-template-columns: 1fr;
  gap: 1rem;
}

@media (min-width: 768px) {
  .form-row {
    grid-template-columns: 1fr 1fr;
  }
}

.input-with-icon {
  position: relative;
}

.input-icon {
  position: absolute;
  left: 0.75rem;
  top: 0.625rem;
  width: 1rem;
  height: 1rem;
  color: #9ca3af;
}

.form-input-with-icon {
  padding-left: 2.5rem;
}

.form-hint {
  font-size: 0.75rem;
  color: #e6e7e8;
  margin: 0;
}

/* Image Upload */
.image-upload-container {
  display: flex;
  align-items: flex-start;
  gap: 1rem;
}

.image-upload-area {
  flex: 1;
}

.upload-zone {
  border: 2px dashed #d1d5db;
  border-radius: 0.5rem;
  padding: 1.5rem;
  text-align: center;
  cursor: pointer;
  transition: border-color 0.2s ease-in-out;
}

.upload-zone:hover {
  border-color: #d6dce5;
}

.file-input {
  display: none;
}

.upload-label {
  cursor: pointer;
}

.upload-icon {
  margin: 0 auto 0.5rem auto;
  width: 3rem;
  height: 3rem;
  color: #dfe3ea;
}

.upload-text {
  margin: 0.5rem 0;
  font-size: 0.875rem;
  color: #dfe2e8;
}

.upload-hint {
  font-size: 0.75rem;
  color: #dfe3ea;
  margin: 0;
}

.image-preview {
  width: 6rem;
  height: 6rem;
}

.preview-image {
  width: 100%;
  height: 100%;
  object-fit: cover;
  border-radius: 0.5rem;
  border: 1px solid #e5e7eb;
}

/* Form Actions */
.form-actions {
  display: flex;
  justify-content: flex-end;
  gap: 1rem;
}

.btn-secondary {
  padding: 0.5rem 1rem;
  border: 1px solid #d1d5db;
  border-radius: 0.375rem;
  color: #d1d5dd;
  background-color: white;
  cursor: pointer;
  font-size: 0.875rem;
}

.btn-secondary:hover {
  background-color: #f9fafb;
}

.btn-primary {
  padding: 0.5rem 1.5rem;
  background-color: #2563eb;
  color: white;
  border: none;
  border-radius: 0.375rem;
  cursor: pointer;
  font-size: 0.875rem;
  min-width: 7.5rem;
}

.btn-primary:hover {
  background-color: #1d4ed8;
}

.btn-primary:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.btn-loading {
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.btn-content {
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.btn-icon {
  width: 1rem;
  height: 1rem;
}

.spinner {
  width: 1rem;
  height: 1rem;
  border: 2px solid white;
  border-top: 2px solid transparent;
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

/* Product Preview */
.preview-card {
  background-color: rgb(2, 25, 10);
  border-radius: 0.5rem;
  box-shadow: 0 1px 3px 0 rgba(0, 0, 0, 0.1);
  padding: 1.5rem;
}

.preview-title {
  font-size: 1.5rem;
  font-weight: 600;
  color: #f7f7f7;
  margin: 0 0 1rem 0;
}

.preview-description {
  color: #c0c3c9;
  margin: 0 0 1rem 0;
}

.preview-content {
  display: flex;
  align-items: flex-start;
  gap: 1rem;
  padding: 1rem;
  border: 1px solid #e5e7eb;
  border-radius: 0.5rem;
}

.preview-image-container {
  width: 4rem;
  height: 4rem;
  background-color: #f3f4f6;
  border-radius: 0.5rem;
  display: flex;
  align-items: center;
  justify-content: center;
}

.preview-product-image {
  width: 100%;
  height: 100%;
  object-fit: cover;
  border-radius: 0.5rem;
}

.preview-placeholder-icon {
  width: 2rem;
  height: 2rem;
  color: #9ca3af;
}

.preview-details {
  flex: 1;
}

.preview-product-name {
  font-weight: 600;
  font-size: 1.5rem;
  color: #e5e7ec;
  margin: 0;
}

.preview-product-description {
  font-size: 0.875rem;
  color: #bfc4ce;
  margin: 0;
}

.preview-product-meta {
  display: flex;
  align-items: center;
  gap: 1rem;
  margin-top: 0.5rem;
}

.preview-price {
  font-size: 0.9rem;
  font-weight: 600;
  color: #ffffff;
}

.preview-stock {
  font-size: 0.9rem;
  color: #dfe3ea;
}

.preview-category {
  font-size: 0.75rem;
  background-color: #dbeafe;
  color: #1e40af;
  padding: 0.25rem 0.5rem;
  border-radius: 0.25rem;
}

/* Animations */
@keyframes spin {
  from {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(360deg);
  }
}

/* Responsive Design */
@media (max-width: 768px) {
  .main-content {
    padding: 1rem;
  }
  
  .header-content {
    padding: 1rem;
  }
  
  .tab-nav {
    gap: 1rem;
  }
  
  .metrics-grid {
    grid-template-columns: 1fr;
  }
  
  .charts-grid {
    grid-template-columns: 1fr;
  }
  
  .search-filters {
    flex-direction: column;
  }
  
  .search-container {
    min-width: auto;
  }
  
  .form-actions {
    flex-direction: column;
  }
  
  .summary-grid {
    grid-template-columns: 1fr;
  }
}
</style>