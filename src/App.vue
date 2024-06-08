<template>
  <div class="container-fluid">
    <div class="row">
      <!-- Sidebar  -->
      <nav id="sidebar" class="col-md-3 col-lg-2 d-md-block bg-dark sidebar text-white">
        <div class="position-sticky">
          <h2 class="text-center py-3 no-cursor">Demo Test</h2>
          <ul class="nav flex-column">
            <li class="nav-item">
              <a class="nav-link text-white" aria-current="page" href="#">
                <i data-feather="user" class="me-1"></i>
                Profile
              </a>
            </li>
            <li class="nav-item">
              <a class="nav-link active text-white" href="#">
                <i data-feather="box" class="me-1"></i>
                Vehicles
              </a>
            </li>
            <li class="nav-item">
              <a class="nav-link text-white" href="#">
                <i data-feather="settings" class="me-1"></i>
                Settings
              </a>
            </li>
          </ul>
        </div>
      </nav>

      <!-- Main content  -->
      <main class="col-md-9 ms-sm-auto col-lg-10 px-md-4">
        <!-- First row -->
        

        <div class="d-flex justify-content-between align-items-center pb-3 mb-3 border-bottom">
          <h1 class="h2 me-3">Vehicles</h1>
          <div class="d-flex align-items-center">
            <img :src="userAvatar" class="img-fluid rounded-circle me-3" style="width: 40px; height: 40px;" alt="User Avatar" >
              <span class="me-3">John Doe</span>
              <div class="dropdown">
                <button class="btn btn-outline-secondary dropdown-toggle" type="button" id="languageDropdown" data-bs-toggle="dropdown" aria-expanded="false">
                  <span class="me-1" data-feather="globe"></span> En
                </button>
                <ul class="dropdown-menu" aria-labelledby="languageDropdown">
                  <li><a class="dropdown-item" href="#">En</a></li>
                  <li><a class="dropdown-item" href="#">Fr</a></li>
                  <li><a class="dropdown-item" href="#">De</a></li>
                </ul>
              </div>
          </div>
        </div>
        
        <!-- Second row -->
        <div class="d-flex justify-content-between align-items-center pb-3 mb-3 border-bottom">
          <input v-model="searchQuery" type="text" class="form-control me-3" placeholder="Search VIN" style="width: 200px;">
          <div class="d-flex align-items-center">
            <label for="vehiclesPerPage" class="me-2">Select vehicles per page:</label>
            <select id="vehiclesPerPage" class="form-select me-3" v-model="vehiclesPerPage" @change="handleVehiclesPerPageChange" style="width: 80px;">
              <option value="3">3</option>
              <option value="6">6</option>
              <option value="9">9</option>
              <option value="12">12</option>
              <option value="15">15</option>
              <option value="18">18</option>
              <option value="21">21</option>
            </select>
            <button class="btn btn-danger" @click="toggleForm">+ ADD VEHICLE</button>
          </div>
        </div>
        
        <!-- Form for adding vehicle -->
        <div v-if="showForm" class="mb-4">
          <h3>Add Vehicle</h3>
          <form @submit.prevent="addVehicle">
            <div class="mb-3">
              
              <label for="vehicleName" class="form-label">Vehicle Name</label>
              <input type="text" class="form-control" id="vehicleName" v-model="newVehicle.vehicle_name" required>
            </div>
            <div class="mb-3">
              <label for="vin" class="form-label">VIN</label>
              <input type="text" class="form-control" id="vin" v-model="newVehicle.vin" required>
            </div>
            <div class="mb-3">
              <label for="anglesCount" class="form-label">Angles Count</label>
              <input type="number" class="form-control" id="anglesCount" v-model="newVehicle.angles_count" required>
            </div>
            <div class="mb-3">
              <label for="photosCount" class="form-label">Photos Count</label>
              <input type="number" class="form-control" id="photosCount" v-model="newVehicle.photos_count" required>
            </div>
            <div class="mb-3">
              <label for="preview" class="form-label">Preview URL</label>
              <input type="url" class="form-control" id="preview" v-model="newVehicle.preview" required>
            </div>
            <button type="submit" class="btn btn-primary">Add Vehicle</button>
            <button type="button" class="btn btn-secondary ms-2" @click="toggleForm">Cancel</button>
          </form>
        </div>

        <div class="row">
          <div v-for="car in displayedCars" :key="car.id" class="col-md-6 col-lg-4 mb-4">
            <div class="card h-100">
              <img :src="car.preview" class="card-img-top" :alt="car.vehicle_name || 'No name'">
              <div class="card-body">
                <h5 class="card-title">{{ car.vehicle_name || 'No name' }}</h5>
                <p class="card-text">{{ car.vin }}</p>
                <p>
                  <span class="badge bg-success">{{ car.angles_count }}/{{ car.photos_count }}</span>
                  <span class="text-muted float-end">{{ daysLeft(car.created_at) }} days left</span>
                </p>
              </div>
            </div>
          </div>
        </div>
        <div class="mt-4 text-center">
          <span>Showing {{ displayedCars.length }} out of {{ cars.length }}</span>
        </div>
      </main>
    </div>
  </div>
</template>

<script>
import userAvatar from '@/assets/Avatar_John_Doe.png';
export default {
  data() {
    return {
      userAvatar: userAvatar,
      cars: [],
      searchQuery: '',
      vehiclesPerPage: 9,
      showForm: false,
      newVehicle: {
        vehicle_name: '',
        vin: '',
        angles_count: 0,
        photos_count: 0,
        preview: ''
      }
    }
  },
  computed: {
    displayedCars() {
      return this.filteredCars.slice(0, this.vehiclesPerPage);
    },
    filteredCars() {
      if (this.searchQuery.trim() === '') {
        return this.cars;
      }
      const query = this.searchQuery.toLowerCase();
      return this.cars.filter(car => 
        car.vin.toLowerCase().includes(query) ||
        (car.vehicle_name && car.vehicle_name.toLowerCase().includes(query))
      );
    }
  },
  mounted() {
    this.getCars();
    this.initializeFeatherIcons();
  },
  methods: {
    async getCars() {
      try {
        let response = await fetch('https://api.caiman-app.de/api/cars-test');
        if (!response.ok) {
          throw new Error(`HTTP error! Status: ${response.status}`);
        }
        let data = await response.json();
        this.cars = data.data;
      } catch (error) {
        console.error('Error fetching cars:', error);
      }
    },
    daysLeft(created_at) {
      const createdDate = new Date(created_at * 1000);
      const now = new Date();
      const diffTime = Math.abs(now - createdDate);
      
      const diffDays = Math.ceil(diffTime / (1000 * 60 * 60 * 24));
      return diffDays; // You can change the logic for the exact 'days left'
    },
    initializeFeatherIcons() {
      this.$nextTick(() => {
        if (window.feather) {
          window.feather.replace();
        }
      });
    },
    toggleForm() {
      this.showForm = !this.showForm;
    },
    addVehicle() {
      this.cars.push({
        ...this.newVehicle,
        id: this.cars.length + 1, // Генерация нового ID
        created_at: Math.floor(new Date().getTime() / 1000)
      });
      this.newVehicle = {
        vehicle_name: '',
        vin: '',
        angles_count: 0,
        photos_count: 0,
        preview: ''
      }; // Очистка формы
      this.toggleForm(); // Закрытие формы
    },
    handleVehiclesPerPageChange() {
    this.getCurrentPageCars();
    },
    getCurrentPageCars() {
      this.displayedCars = this.filteredCars.slice(0, this.vehiclesPerPage);
    }
  },
  updated() {
    this.initializeFeatherIcons();
  }
}
</script>

<!-- Integrate Bootstrap CSS -->
<style>
@import "https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/css/bootstrap.min.css";
@import "https://unpkg.com/feather-icons/dist/feather.css";

.container-fluid {
  margin-top: 20px;
}

#sidebar {
  height: 100vh;
  position: fixed;
}

#sidebar .nav-link {
  color: white;
}

#sidebar .nav-link.active {
  background-color: rgba(255, 255, 255, 0.1);
}

#sidebar .nav-link:hover {
  background-color: rgba(255, 255, 255, 0.1);
}

.no-cursor {
  cursor: default;
}
</style>
