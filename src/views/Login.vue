<template>
  <v-container fill-height fluid>
    <v-row align="center" justify="center">
      <v-card
        class="justify-center"
        max-width="600px"
        min-width="360px"
        elevation="24"
      >
        <v-tabs
          v-model="tab"
          show-arrows
          background-color="primary"
          icons-and-text
          grow
        >
          <v-tabs-slider color="accent"></v-tabs-slider>
          <v-tab :key="0">
            <v-icon>mdi-account-key</v-icon>
            <div class="body-1">Login</div>
          </v-tab>
          <v-tab :key="1">
            <v-icon>mdi-account-plus</v-icon>
            <div class="body-1">Sign Up</div>
          </v-tab>
          <v-tab-item>
            <v-card class="px-4" width="600px">
              <v-card-text>
                <v-form ref="loginForm" v-model="valid" lazy-validation>
                  <v-text-field
                    v-model="loginEmail"
                    :rules="emailRules"
                    label="E-mail"
                    required
                  ></v-text-field>
                  <v-text-field
                    v-model="loginPassword"
                    :append-icon="show1 ? 'eye' : 'eye-off'"
                    :rules="[rules.required, rules.min]"
                    :type="show1 ? 'text' : 'password'"
                    name="input-10-1"
                    label="Password"
                    hint="At least 8 characters"
                    ref="email"
                    counter
                    @click:append="show1 = !show1"
                  ></v-text-field>
                  <v-card-actions class="pt-4">
                    <v-spacer></v-spacer>
                    <v-dialog v-model="dialog" persistent max-width="600px">
                      <template v-slot:activator="{ on, attrs }">
                        <v-btn color="secondary" text v-bind="attrs" v-on="on">
                          Forget Password
                        </v-btn>
                      </template>
                      <v-card>
                        <v-card-title>
                          <span class="text-h5">Forget Password?</span>
                        </v-card-title>
                        <v-card-text>
                          <v-form
                            ref="forgetForm"
                            v-model="valid"
                            lazy-validation
                          >
                            Enter your email below and an email will be sent to
                            you to reset your password.
                            <v-text-field
                              class="mt-4"
                              v-model="forgetEmail"
                              :rules="emailRules"
                              label="E-mail"
                              required
                            ></v-text-field>
                            <v-card-actions class="pt-4">
                              <v-spacer></v-spacer>
                              <v-btn
                                color="secondary"
                                text
                                @click="dialog = false"
                              >
                                Close
                              </v-btn>
                              <v-btn
                                :disabled="!valid"
                                color="secondary"
                                @click="validateForget"
                              >
                                Send Email
                              </v-btn>
                            </v-card-actions>
                          </v-form>
                        </v-card-text>
                      </v-card>
                    </v-dialog>
                    <v-btn
                      :disabled="!valid"
                      color="secondary"
                      @click="validateLogin"
                    >
                      Login
                    </v-btn>
                  </v-card-actions>
                </v-form>
              </v-card-text>
            </v-card>
          </v-tab-item>
          <v-tab-item>
            <v-card class="px-4" width="600px">
              <v-card-text>
                <v-form ref="signUpForm" v-model="valid" lazy-validation>
                  <v-text-field
                    v-model="email"
                    :rules="emailRules"
                    label="E-mail"
                    required
                  ></v-text-field>
                  <v-text-field
                    v-model="password"
                    :append-icon="show1 ? 'mdi-eye' : 'mdi-eye-off'"
                    :rules="[rules.required, rules.min]"
                    :type="show1 ? 'text' : 'password'"
                    name="input-10-1"
                    label="Password"
                    hint="At least 8 characters"
                    counter
                    @click:append="show1 = !show1"
                  ></v-text-field>
                  <v-text-field
                    block
                    v-model="verify"
                    :append-icon="show1 ? 'mdi-eye' : 'mdi-eye-off'"
                    :rules="[rules.required, rules.min]"
                    :type="show1 ? 'text' : 'password'"
                    name="input-10-1"
                    label="Confirm Password"
                    counter
                    @click:append="show1 = !show1"
                  ></v-text-field>
                  <v-card-actions class="pt-4">
                    <v-spacer></v-spacer>
                    <v-btn
                      :disabled="!valid"
                      color="secondary"
                      @click="validateSignUp"
                      >Register</v-btn
                    >
                  </v-card-actions>
                </v-form>
              </v-card-text>
            </v-card>
          </v-tab-item>
        </v-tabs>
      </v-card>
    </v-row>
  </v-container>
</template>

<script>
import {
  signInWithEmailAndPassword,
  createUserWithEmailAndPassword,
  sendPasswordResetEmail,
} from "firebase/auth"
import { auth } from "../firebase"

export default {
  name: "Login",
  methods: {
    validateForget() {
      if (this.$refs.forgetForm.validate()) {
        sendPasswordResetEmail(auth, this.forgetEmail)
          .then(() => {
            this.$root.toast.show({ message: "Email sent!" })
            this.dialog = false
          })
          .catch((error) => {
            this.$root.toast.show({ message: error.message })
          })
      } else {
        this.$root.toast.show({ message: "Not sent!" })
      }
    },
    validateLogin() {
      if (this.$refs.loginForm.validate()) {
        signInWithEmailAndPassword(auth, this.loginEmail, this.loginPassword)
          .then(() => {
            this.$root.toast.show({ message: "Succesfully signed in!" })
            this.$router.push("/dashboard")
          })
          .catch((error) => {
            this.$root.toast.show({ message: error.message })
          })
      }
    },
    validateSignUp() {
      if (this.$refs.signUpForm.validate()) {
        if (this.password === this.verify) {
          createUserWithEmailAndPassword(auth, this.email, this.password)
            .then(() => {
              this.$root.toast.show({
                message: "Succesfully created an account!",
              })
              this.$router.push("/dashboard")
            })
            .catch((error) => {
              this.$root.toast.show({ message: error.message })
            })
        } else {
          this.$root.toast.show({ message: "Passwords do not match." })
        }
      }
    },
    reset() {
      this.$refs.form.reset()
    },
    resetValidation() {
      this.$refs.form.resetValidation()
    },
  },
  data: () => ({
    tab: 0,
    tabs: [
      { name: "Login", icon: "mdi-account-key" },
      { name: "Register", icon: "mdi-account-plus" },
    ],
    valid: true,
    email: "",
    password: "",
    verify: "",
    loginEmail: "",
    loginPassword: "",
    forgetEmail: "",
    dialog: false,
    emailRules: [
      (v) => !!v || "Required",
      (v) => /.+@.+\..+/.test(v) || "E-mail must be valid",
    ],
    show1: false,
    rules: {
      required: (value) => !!value || "Required.",
      min: (v) => (v && v.length >= 8) || "Min 8 characters",
    },
  }),
  metaInfo: {
    title: "Login",
  },
}
</script>