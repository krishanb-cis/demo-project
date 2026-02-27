<template>
  <div class="container">
    <h1>📝 CRUD App</h1>

    <!-- authentication -->
    <div v-if="!user" class="auth-section">
      <h2>{{ isRegistering ? 'Register' : 'Login' }}</h2>
      <form @submit.prevent="handleAuth">
        <div class="form-group">
          <label for="email">Email:</label>
          <input v-model="authForm.email" type="email" id="email" required />
        </div>
        <div class="form-group">
          <label for="password">Password:</label>
          <input v-model="authForm.password" type="password" id="password" required />
        </div>
        <div class="form-buttons">
          <button type="submit" class="btn btn-primary">
            {{ isRegistering ? 'Register' : 'Login' }}
          </button>
          <button type="button" class="btn btn-secondary" @click="isRegistering = !isRegistering">
            {{ isRegistering ? 'Have an account? Login' : "Need an account? Register" }}
          </button>
        </div>
      </form>
      <div v-if="authError" class="error-message">{{ authError }}</div>
    </div>

    <!-- logout / welcome -->
    <div v-else class="welcome">
      <p>Welcome, {{ user.email }} <button @click="logout" class="btn btn-secondary btn-small">Logout</button></p>
    </div>

    <!-- main CRUD UI (shown when authenticated) -->
    <div v-if="user" class="crud-section">
      <div class="form-section">
        <h2>{{ editingId ? 'Edit Item' : 'Add New Item' }}</h2>
        <form @submit.prevent="saveItem">
          <div class="form-group">
            <label for="title">Title:</label>
            <input 
              v-model="form.title" 
              type="text" 
              id="title" 
              placeholder="Enter title"
              required
            >
          </div>
          
          <div class="form-group">
            <label for="description">Description:</label>
            <textarea 
              v-model="form.description" 
              id="description" 
              placeholder="Enter description"
              required
            ></textarea>
          </div>
          
          <div class="form-group">
            <label for="status">Status:</label>
            <select v-model="form.status" id="status">
              <option value="pending">Pending</option>
              <option value="completed">Completed</option>
            </select>
          </div>
          
          <div class="form-buttons">
            <button type="submit" class="btn btn-primary">
              {{ editingId ? 'Update' : 'Add' }} Item
            </button>
            <button 
              v-if="editingId" 
              type="button" 
              class="btn btn-secondary"
              @click="cancelEdit"
            >
              Cancel
            </button>
          </div>
        </form>
      </div>
  
      <div class="items-section">
        <h2>Items List ({{ items.length }})</h2>
        
        <div v-if="loading" class="loading">Loading items...</div>
        <div v-else-if="items.length === 0" class="empty">No items yet. Create one!</div>
        
        <div v-else class="items-grid">
          <div v-for="item in items" :key="item._id" class="item-card">
            <div class="item-header">
              <h3>{{ item.title }}</h3>
              <span :class="['status', item.status]">{{ item.status }}</span>
            </div>
            
            <p class="item-description">{{ item.description }}</p>
            
            <div class="item-meta">
              <small>{{ formatDate(item.createdAt) }}</small>
            </div>
            
            <div class="item-actions">
              <button class="btn btn-edit" @click="editItem(item)">Edit</button>
              <button class="btn btn-delete" @click="deleteItem(item._id)">Delete</button>
            </div>
          </div>
        </div>
      </div>

      <div v-if="error" class="error-message">{{ error }}</div>
    </div>
  </div>
</template>

<script>
import { ref, onMounted } from 'vue'
import axios from 'axios'

const API_URL = 'http://localhost:5000/api'

// configure axios to include token if present
const token = localStorage.getItem('token')
if (token) {
  axios.defaults.headers.common['Authorization'] = `Bearer ${token}`
}

export default {
  name: 'App',
  setup() {
    // authentication state
    const user = ref(JSON.parse(localStorage.getItem('user')) || null)
    const authError = ref('')
    const isRegistering = ref(false)
    const authForm = ref({ email: '', password: '' })

    // items state
    const items = ref([])
    const loading = ref(false)
    const error = ref('')
    const editingId = ref(null)
    const form = ref({
      title: '',
      description: '',
      status: 'pending'
    })

    const saveItem = async () => {
      try {
        error.value = ''
        if (editingId.value) {
          const response = await axios.put(
            `${API_URL}/items/${editingId.value}`,
            form.value
          )
          const index = items.value.findIndex(item => item._id === editingId.value)
          if (index !== -1) items.value[index] = response.data
          editingId.value = null
        } else {
          const response = await axios.post(`${API_URL}/items`, form.value)
          items.value.unshift(response.data)
        }
        resetForm()
      } catch (err) {
        error.value = 'Failed to save item: ' + err.message
        console.error(err)
      }
    }

    const fetchItems = async () => {
      if (!user.value) return
      try {
        loading.value = true
        error.value = ''
        const response = await axios.get(`${API_URL}/items`)
        items.value = response.data
      } catch (err) {
        error.value = 'Failed to fetch items: ' + err.message
        console.error(err)
      } finally {
        loading.value = false
      }
    }

    const editItem = (item) => {
      editingId.value = item._id
      form.value = {
        title: item.title,
        description: item.description,
        status: item.status
      }
      window.scrollTo({ top: 0, behavior: 'smooth' })
    }

    const deleteItem = async (id) => {
      if (confirm('Are you sure you want to delete this item?')) {
        try {
          error.value = ''
          await axios.delete(`${API_URL}/items/${id}`)
          items.value = items.value.filter(item => item._id !== id)
        } catch (err) {
          error.value = 'Failed to delete item: ' + err.message
          console.error(err)
        }
      }
    }

    const cancelEdit = () => {
      editingId.value = null
      resetForm()
    }

    const resetForm = () => {
      form.value = {
        title: '',
        description: '',
        status: 'pending'
      }
    }

    const formatDate = (dateString) => {
      return new Date(dateString).toLocaleDateString('en-US', {
        year: 'numeric',
        month: 'short',
        day: 'numeric',
        hour: '2-digit',
        minute: '2-digit'
      })
    }

    const handleAuth = async () => {
      authError.value = ''
      const endpoint = isRegistering.value ? 'register' : 'login'
      try {
        const res = await axios.post(`${API_URL}/auth/${endpoint}`, authForm.value)
        user.value = res.data.user
        localStorage.setItem('user', JSON.stringify(user.value))
        localStorage.setItem('token', res.data.token)
        axios.defaults.headers.common['Authorization'] = `Bearer ${res.data.token}`
        authForm.value = { email: '', password: '' }
        fetchItems()
      } catch (err) {
        authError.value = err.response?.data?.message || err.message
      }
    }

    const logout = () => {
      user.value = null
      items.value = []
      localStorage.removeItem('user')
      localStorage.removeItem('token')
      delete axios.defaults.headers.common['Authorization']
    }

    onMounted(() => {
      if (user.value) fetchItems()
    })

    return {
      // auth
      user,
      authError,
      isRegistering,
      authForm,
      handleAuth,
      logout,
      // items
      items,
      loading,
      error,
      editingId,
      form,
      saveItem,
      editItem,
      deleteItem,
      cancelEdit,
      formatDate
    }
  }
}
</script>

<style scoped>
.container {
  background: white;
  border-radius: 12px;
  padding: 30px;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.3);
}

h1 {
  text-align: center;
  color: #333;
  margin-bottom: 30px;
  font-size: 2.5em;
}

h2 {
  color: #667eea;
  margin-bottom: 20px;
  font-size: 1.5em;
}

.form-section {
  background: #f8f9fa;
  padding: 25px;
  border-radius: 8px;
  margin-bottom: 30px;
  border-left: 4px solid #667eea;
}

.form-group {
  margin-bottom: 15px;
}

label {
  display: block;
  margin-bottom: 8px;
  color: #333;
  font-weight: 500;
}

input, textarea, select {
  width: 100%;
  padding: 10px 12px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-family: inherit;
  font-size: 1em;
  transition: border-color 0.3s;
}

input:focus, textarea:focus, select:focus {
  outline: none;
  border-color: #667eea;
  box-shadow: 0 0 0 2px rgba(102, 126, 234, 0.1);
}

textarea {
  resize: vertical;
  min-height: 80px;
}

.form-buttons {
  display: flex;
  gap: 10px;
  margin-top: 20px;
}

.btn {
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-weight: 500;
  transition: all 0.3s;
  font-size: 0.95em;
}

.btn-primary {
  background: #667eea;
  color: white;
  flex: 1;
}

.btn-primary:hover {
  background: #5568d3;
  transform: translateY(-2px);
  box-shadow: 0 5px 15px rgba(102, 126, 234, 0.4);
}

.btn-secondary {
  background: #6c757d;
  color: white;
}

.btn-secondary:hover {
  background: #5a6268;
}

.btn-edit {
  background: #28a745;
  color: white;
  padding: 8px 16px;
  font-size: 0.9em;
}

.btn-edit:hover {
  background: #218838;
}

.btn-delete {
  background: #dc3545;
  color: white;
  padding: 8px 16px;
  font-size: 0.9em;
}

.btn-delete:hover {
  background: #c82333;
}

.items-section {
  margin-top: 40px;
}

/* authentication styles */
.auth-section {
  background: #f8f9fa;
  padding: 25px;
  border-radius: 8px;
  margin-bottom: 30px;
  border-left: 4px solid #28a745;
}

.welcome {
  margin-bottom: 20px;
}

.btn-small {
  padding: 4px 8px;
  font-size: 0.8em;
}

.loading, .empty {
  text-align: center;
  padding: 40px 20px;
  color: #999;
  font-size: 1.1em;
}

.items-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: 20px;
}

.item-card {
  background: #fff;
  border: 1px solid #e0e0e0;
  border-radius: 8px;
  padding: 20px;
  transition: all 0.3s;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.item-card:hover {
  box-shadow: 0 8px 16px rgba(0, 0, 0, 0.15);
  transform: translateY(-4px);
}

.item-header {
  display: flex;
  justify-content: space-between;
  align-items: start;
  margin-bottom: 10px;
  gap: 10px;
}

.item-header h3 {
  color: #333;
  margin: 0;
  flex: 1;
}

.status {
  padding: 4px 12px;
  border-radius: 20px;
  font-size: 0.85em;
  font-weight: 500;
  white-space: nowrap;
}

.status.pending {
  background: #fff3cd;
  color: #856404;
}

.status.completed {
  background: #d4edda;
  color: #155724;
}

.item-description {
  color: #666;
  margin: 12px 0;
  line-height: 1.5;
}

.item-meta {
  color: #999;
  font-size: 0.85em;
  margin-bottom: 15px;
}

.item-actions {
  display: flex;
  gap: 10px;
}

.error-message {
  background: #f8d7da;
  border: 1px solid #f5c6cb;
  color: #721c24;
  padding: 15px;
  border-radius: 4px;
  margin-top: 20px;
}

@media (max-width: 768px) {
  .container {
    padding: 20px;
  }

  h1 {
    font-size: 2em;
  }

  .items-grid {
    grid-template-columns: 1fr;
  }

  .form-buttons {
    flex-direction: column;
  }

  .item-actions {
    flex-direction: column;
  }

  .btn-edit, .btn-delete {
    width: 100%;
  }
}
</style>
