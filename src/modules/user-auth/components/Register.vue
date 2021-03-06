<template>
  <v-container v-if="isLogin" fluid fill-height class="grey lighten-3">
    <v-layout align-center justify-center>
      <v-flex xs12 sm8 md4>
        <v-card class="elevation-12">
          <v-card-text>
            Sesión Iniciada como:
            <strong>{{getUser.username}}</strong>
          </v-card-text>
        </v-card>
      </v-flex>
    </v-layout>
  </v-container>

  <v-container v-else-if="isAccountCreated" fluid fill-height class="grey lighten-3">

    <v-layout align-center justify-center>
      <v-flex xs12 sm8 md4>
        <v-card class="elevation-12">
          <v-card-text>
            <v-alert
              :value="true"
              color="success"
              icon="check_circle"
              outline
            >
              Hemos enviado un correo a <strong>{{form.email}}</strong> para validar y finalizar la creación de su cuenta.
            </v-alert>

          </v-card-text>
        </v-card>
      </v-flex>
    </v-layout>
  </v-container>

  <v-container v-else fluid fill-height class="grey lighten-3">

    <v-layout align-center justify-center>
      <v-flex xs12 sm8 md4>
        <v-card class="elevation-12">
          <v-toolbar dark color="secondary">
            <v-toolbar-title>Crear Cuenta</v-toolbar-title>
            <v-spacer></v-spacer>


          </v-toolbar>
          <v-card-text>
            <v-form ref="form" autocomplete="off" v-model="valid">


              <v-text-field prepend-icon="account_box"
                            name="name"
                            label="Nombre y Apellido"
                            type="text"
                            v-model="form.name"
                            :rules="validations.name"
                            placeholder="Nombre y Apellido"

                            :error="errors.name.length?true:false"
                            :error-messages="errors.name"


                            required
              >

              </v-text-field>

              <v-text-field prepend-icon="person"
                            name="username"
                            label="Usuario"
                            type="text"
                            v-model="form.username"
                            :rules="validations.username"
                            placeholder="Usuario"
                            autocomplete="new-password"

                            :error="errors.username.length?true:false"
                            :error-messages="errors.username"

                            required
              >

              </v-text-field>

              <v-text-field prepend-icon="email"
                            name="email"
                            label="Email"
                            type="text"
                            v-model="form.email"
                            :rules="validations.email"
                            placeholder="Email"

                            :error="errors.email.length?true:false"
                            :error-messages="errors.email"

                            required
              >

              </v-text-field>

              <v-text-field prepend-icon="email"
                            name="email_verify"
                            label="Repetir Email"
                            type="text"
                            v-model="form.email_verify"
                            :rules="validations.email_verify"
                            placeholder="Repetir Email"
                            required
                            onPaste="return false"
                            :error="emailMatchError == '' ? false : true"
                            :error-messages="emailMatchError"

              >

              </v-text-field>

              <v-text-field prepend-icon="phone"
                            name="phone"
                            label="Telefono"
                            type="text"
                            v-model="form.phone"
                            :rules="validations.phone"
                            placeholder="Telefono"

                            :error="errors.phone.length?true:false"
                            :error-messages="errors.phone"

                            required
              >

              </v-text-field>

              <v-text-field id="password"
                            prepend-icon="lock"
                            name="password"
                            label="Contraseña"
                            type="password"
                            v-model="form.password"
                            :rules="validations.password"
                            placeholder="Contraseña"
                            autocomplete="new-password"
                            ref="password"
                            :error="errors.password.length?true:false"
                            :error-messages="errors.password"


                            required
              >
              </v-text-field>
                <v-text-field id="password_verify"
                              prepend-icon="lock"
                              name="password_verify"
                              label="Repetir Contraseña"
                              type="password"
                              v-model="form.password_verify"
                              placeholder="Repetir Contraseña"
                              autocomplete="new-password"

                              :error="passwordMatchError == '' ? false : true"
                              :error-messages="passwordMatchError"


                              required
                >

                </v-text-field>
            </v-form>
          </v-card-text>
          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn :loading="getAuthLoading" color="primary" flat dark @click="submit">Crear Cuenta</v-btn>
          </v-card-actions>
        </v-card>

      </v-flex>
    </v-layout>
  </v-container>
</template>

<script>

  import {mapActions, mapGetters} from 'vuex'

  export default {
    name: "Register",
    data: () => ({
        valid: true,
        isAccountCreated: false,
        form: {
          name: null,
          username: null,
          password: null,
          password_verify: null,
          email: null,
          email_verify: null,
          phone: null,
        },
        errors: {
          name: [],
          username: [],
          password: [],
          password_verify: [],
          email: [],
          phone: [],
        },
        validations: {
          name: [
            v => !!v || 'Nombre y Apellido es requerido',
          ],
          username: [
            v => !!v || 'Usuario es requerido',
          ],
          phone: [
            v => !!v || 'Telefono es requerido',
          ],
          email: [
            v => !!v || 'Email es requerido',
            v => /.+@.+/.test(v) || 'Email debe tener un formato valido '
          ],
          password: [
            v => !!v || 'Contraseña es requerido',
          ]
        }
      }
    ),
    computed: {
      ...mapGetters([
        'getUser',
        'isLogin',
        'getAuthLoading'
      ]),
      passwordMatchError () {
        return (this.form.password === this.form.password_verify) ? '' : 'Contraseña no coincide'
      },
      emailMatchError () {
        return (this.form.email === this.form.email_verify) ? '' : 'Email no coincide'
      }
    },
    methods: {
      resetValidation: function () {
        this.valid = true
        //  this.$refs.form.resetValidation()
        this.errors = {
          name: [],
          username: [],
          password: [],
          email: [],
          phone: [],
        }
      },
      submit: function () {

        this.resetValidation()

        if (this.$refs.form.validate()) {
          this.$store.commit('SET_AUTH_LOADING', true)
          this.register(this.form).then((response) => {
            if (response.data.status) {
              //TODO confirmar creacion de cuenta
              this.isAccountCreated = true
            } else {
              this.errors = Object.assign({}, this.errors, response.data.errors);
            }
            this.$store.commit('SET_AUTH_LOADING', false)
          })
        }
      },
      ...mapActions([
        'register',
      ]),
    }
  }
</script>

<style scoped>
  a {
    text-decoration: none;
  }
</style>
