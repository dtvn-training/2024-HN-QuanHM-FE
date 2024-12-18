<template>
  <div class="content-wrapper">
    <v-container class="fill-height" fluid>
      <v-row align="center" justify="center">
        <v-col cols="12" sm="8" md="6" lg="4">
          <v-card class="login-card">
            <v-card-title class="text-h4 font-weight-bold text-center my-4">
              Welcome Back
            </v-card-title>
            <v-card-text>
              <v-form @submit.prevent="handleLogin" ref="form">
                <v-text-field
                  v-model="email"
                  :rules="[rules.required, rules.email]"
                  label="Email"
                  prepend-inner-icon="mdi-email"
                  variant="outlined"
                  class="mb-4"
                ></v-text-field>
                <v-text-field
                  v-model="password"
                  label="Password"
                  prepend-inner-icon="mdi-lock"
                  :append-inner-icon="showPassword ? 'mdi-eye' : 'mdi-eye-off'"
                  @click:append-inner="showPassword = !showPassword"
                  :type="showPassword ? 'text' : 'password'"
                  variant="outlined"
                  class="mb-4"
                ></v-text-field>
                <v-btn
                  block
                  color="primary"
                  size="large"
                  type="submit"
                  :loading="isLoading"
                >
                  Sign In
                </v-btn>
                <v-btn @click="loginWithGoogle"> Login with Google </v-btn>
              </v-form>
            </v-card-text>
            <v-card-actions class="justify-center">
              <v-btn variant="text" @click="handleResetPassword">
                Forgot Password?
              </v-btn>
              <v-btn
                @click="this.$router.push({ name: 'Register' })"
                variant="text"
              >
                Sign Up
              </v-btn>
            </v-card-actions>
          </v-card>
        </v-col>
      </v-row>
    </v-container>
  </div>
  <v-snackbar v-model="snackbar.show" :color="snackbar.color" location="top">
    {{ snackbar.text }}
  </v-snackbar>
</template>

<script setup>
import { ref, reactive } from "vue";
import axios from "axios";
import { useRouter } from "vue-router";
import { useStore } from "vuex";

const store = useStore();
const form = ref(null);
const email = ref("");
const password = ref("");
const showPassword = ref(false);
const isLoading = ref(false);
const router = useRouter();
const snackbar = reactive({
  show: false,
  text: "",
  color: "info",
});

const rules = {
  required: (v) => !!v || "This field is required",
  email: (v) => /.+@.+\..+/.test(v) || "Please enter a valid email",
  minLength: (v) => v.length >= 6 || "Password must be at least 8 characters",
};

const showSnackbar = (text, color = "info") => {
  snackbar.text = text;
  snackbar.color = color;
  snackbar.show = true;
};

const useAuth = () => {
  const login = async (email, password) => {
    try {
      isLoading.value = true;
      // Replace with your actual API endpoint
      const response = await axios.post("http://localhost:8080/log-in", {
        email,
        password,
      });
      const token = response.data.token;
      localStorage.setItem("token", token);
      console.log(store);
      // store.dispatch("login", token);
      // console.log(store.getters.email);
      showSnackbar("Login successful", "success");
      router.push({ name: "Home" }); // Redirect after successful login
      // Add your post-login logic here (e.g., redirect to dashboard)
    } catch (error) {
      console.error("Login error:", error);
      showSnackbar(error.response?.data?.message || "Login failed", "error");
    } finally {
      isLoading.value = false;
    }
  };

  return { login };
};

const { login } = useAuth();

const loginWithGoogle = async () => {
  try {
    const response = await axios.get("http://localhost:8080/google/login-url");
    console.log(response.data);
    window.location.href = response.data;
  } catch (error) {
    console.error("Password reset error:", error);
    showSnackbar(
      error.response?.data?.message || "Password reset failed",
      "error"
    );
  }
};
const handleLogin = async () => {
  const { valid } = await form.value.validate();
  if (valid) {
    await login(email.value, password.value);
  }
};

const handleResetPassword = () => {
  router.push({ name: "ForgotPassword" });
};
</script>

<style scoped>
.login-container {
  position: relative;
  width: 100%;
  height: 100vh;
  overflow: hidden;
}

.background-image {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-image: url("https://images4.alphacoders.com/134/1341416.png");
  background-size: cover;
  background-position: center;
  background-repeat: no-repeat;
}

.content-wrapper {
  position: relative;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.4);
  backdrop-filter: blur(1px);
  z-index: 1;
}

.login-card {
  background: rgba(255, 255, 255, 0.7) !important;
  backdrop-filter: blur(10px);
  border-radius: 16px !important;
  padding: 2rem;
  box-shadow: 0 4px 30px rgba(0, 0, 0, 0.1) !important;
  border: 1px solid rgba(255, 255, 255, 0.3) !important;
}

.v-text-field ::v-deep .v-field__outline__start,
.v-text-field ::v-deep .v-field__outline__end {
  border-color: rgba(255, 255, 255, 0.7) !important;
}

.v-text-field ::v-deep .v-field__outline__notch {
  border-color: rgba(255, 255, 255, 0.7) !important;
}

.v-text-field ::v-deep .v-field__outline {
  color: rgba(0, 0, 0, 0.6) !important;
}

.v-text-field ::v-deep .v-field__input {
  color: rgba(0, 0, 0, 0.87) !important;
}

.v-text-field ::v-deep .v-label {
  color: rgba(0, 0, 0, 0.6) !important;
}

@media (max-width: 600px) {
  .login-card {
    padding: 1rem;
  }
}
</style>
