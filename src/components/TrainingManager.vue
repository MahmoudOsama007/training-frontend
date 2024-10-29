<template>
  <div class="all">
    <div class="container-fluid py-4 bg-light min-vh-100">
      <div class="row justify-content-center">
        <div class="col-12 col-md-8 col-lg-6">
          <!-- Header -->
          <div class="card shadow-sm mb-4 max-width">
            <div class="card-body text-center">
              <h1 class="display-6 mb-0">Training Manager</h1>
            </div>
          </div>

          <!-- Form Card -->
          <div class="card shadow-sm mb-4 mt-5 max-width">
            <div class="card-body">
              <form @submit.prevent="addTraining">
                <div class="mb-3">
                  <label for="trainingName" class="form-label"
                    >Training Name</label
                  >
                  <div class="input-group">
                    <input
                      type="text"
                      id="trainingName"
                      v-model="newTrainingName"
                      class="form-control"
                      placeholder="Enter training name"
                      required
                    />
                    <button type="submit" class="btn btn-primary">
                      <i class="bi bi-plus-lg"></i> Add Training
                    </button>
                  </div>
                </div>
              </form>
            </div>
          </div>

          <!-- Alert Messages -->
          <div
            v-if="error"
            class="alert alert-danger alert-dismissible fade show"
            role="alert"
          >
            {{ error }}
            <button
              type="button"
              class="btn-close"
              @click="error = null"
            ></button>
          </div>

          <div
            v-if="success"
            class="alert alert-success alert-dismissible fade show"
            role="alert"
          >
            {{ success }}
            <button
              type="button"
              class="btn-close"
              @click="success = null"
            ></button>
          </div>

          <!-- Trainings Table Card -->
          <div class="card shadow-sm mt-5 max-width">
            <div class="card-body">
              <div v-if="loading" class="text-center py-4">
                <div class="spinner-border text-primary" role="status">
                  <span class="visually-hidden">Loading...</span>
                </div>
              </div>

              <div v-else>
                <table v-if="trainings.length" class="table table-hover">
                  <thead class="table-light">
                    <tr>
                      <th scope="col">#</th>
                      <th scope="col">Training Name</th>
                      <th scope="col">Actions</th>
                    </tr>
                  </thead>
                  <tbody>
                    <tr v-for="training in trainings" :key="training.id">
                      <td>{{ training.id }}</td>
                      <td>
                        <div
                          v-if="editingId === training.id"
                          class="input-group"
                        >
                          <input
                            type="text"
                            v-model="editedName"
                            class="form-control form-control-sm"
                            @keyup.enter="updateTraining(training.id)"
                          />
                          <button
                            class="btn btn-sm btn-success"
                            @click="updateTraining(training.id)"
                          >
                            <i class="bi bi-check">update</i>
                          </button>
                          <button
                            class="btn btn-sm btn-secondary"
                            @click="cancelEdit"
                          >
                            <i class="bi bi-x">cancel</i>
                          </button>
                        </div>
                        <span v-else>{{ training.name }}</span>
                      </td>
                      <td>
                        <button
                          class="btn btn-sm btn-outline-primary me-2"
                          @click="startEdit(training)"
                        >
                          <i class="bi bi-pencil">Edit</i>
                        </button>
                        <button
                          class="btn btn-sm btn-outline-danger"
                          @click="deleteTraining(training.id)"
                        >
                          <i class="bi bi-trash">Delete</i>
                        </button>
                      </td>
                    </tr>
                  </tbody>
                </table>
                <div v-else class="text-center py-4 text-muted">
                  <i class="bi bi-inbox-fill fs-1"></i>
                  <p class="mt-2">No trainings found</p>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
const API_URL = "https://localhost:7196/api/TrainingInitiative";

export default {
  data() {
    return {
      trainings: [],
      newTrainingName: "",
      error: null,
      success: null,
      loading: true,
      editingId: null,
      editedName: "",
    };
  },
  methods: {
    async fetchTrainings() {
      this.loading = true;
      try {
        const response = await fetch(API_URL, {
          method: "GET",
          headers: {
            Accept: "application/json",
            "Content-Type": "application/json",
          },
          credentials: "include",
        });

        if (!response.ok) {
          const errorData = await response.json().catch(() => null);
          throw new Error(
            errorData?.message || `Server error: ${response.status}`
          );
        }

        this.trainings = await response.json();
        this.error = null;
      } catch (error) {
        this.handleError(error);
      } finally {
        this.loading = false;
      }
    },
    async addTraining() {
      try {
        const response = await fetch(API_URL, {
          method: "POST",
          headers: {
            Accept: "application/json",
            "Content-Type": "application/json",
          },
          credentials: "include",
          body: JSON.stringify({ name: this.newTrainingName }),
        });

        if (!response.ok) {
          const errorData = await response.json().catch(() => null);
          throw new Error(
            errorData?.message || `Server error: ${response.status}`
          );
        }

        this.success = "Training added successfully!";
        this.newTrainingName = "";
        await this.fetchTrainings();
        this.error = null;
      } catch (error) {
        this.handleError(error);
      }
    },
    // async deleteTraining(id) {
    //   try {
    //     const response = await fetch(`${API_URL}/${id}`, {
    //       method: "DELETE",
    //       headers: {
    //         Accept: "application/json",
    //         "Content-Type": "application/json",
    //       },
    //       credentials: "include",
    //     });
    //     if (!response.ok) {
    //       const errorData = await response.json().catch(() => null);
    //       throw new Error(
    //         errorData?.message || `Server error: ${response.status}`
    //       );
    //     }

    //     this.fetchTrainings();
    //   } catch (error) {
    //     this.handleError(error);
    //   }
    // },
    handleError(error) {
      if (error.name === "TypeError" && error.message === "Failed to fetch") {
        this.error =
          "Unable to connect to the server. Please check if the API is running.";
      } else {
        this.error = `Error: ${error.message}`;
      }
      console.error("Error:", error);
      this.trainings = [];
    },
    startEdit(training) {
      this.editingId = training.id;
      this.editedName = training.name;
    },
    cancelEdit() {
      this.editingId = null;
      this.editedName = "";
    },
    async updateTraining(id) {
      try {
        const response = await fetch(`${API_URL}/${id}`, {
          method: "PUT",
          headers: {
            Accept: "application/json",
            "Content-Type": "application/json",
          },
          credentials: "include",
          body: JSON.stringify({ id, name: this.editedName }),
        });

        if (!response.ok) {
          const errorData = await response.json().catch(() => null);
          throw new Error(
            errorData?.message || `Server error: ${response.status}`
          );
        }

        this.success = "Training updated successfully!";
        this.editingId = null;
        this.editedName = "";
        await this.fetchTrainings();
      } catch (error) {
        this.handleError(error);
      }
    },
  },
  created() {
    this.fetchTrainings();
  },
};
</script>

<style scoped>
.bg {
  background: linear-gradient(to right, #9356d4, #4136a0);
}
.max-width {
  max-width: 1200px;
}
.container-fluid {
  /* max-width: 1200px; */

  background: linear-gradient(
    to right,
    #9356d4,
    #4136a0
  ); /* Adjust colors as needed */
}
</style>
