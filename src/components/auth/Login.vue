<template>
  <div class="app flex-row align-items-center">
    <div class="container">
      <b-row class="justify-content-center">
        <b-col md="8">
          <b-card-group>
            <b-card no-body class="p-4">
              <b-card-body>
                <h1>Iniciar Sesión</h1>
                <div class="alert alert-danger" role="alert" :key="key" v-for="(error_message, key) in errors">
                  {{error_message}}
                </div> 
                <p class="text-muted">Aquí podras hacer seguimiento de tu estado de salud</p>
                  <div v-if="!confirmationCodeStage">
                    <b-input-group class="mb-12">
                      <vue-tel-input type="text" class="form-control" placeholder="Número de Teléfono" v-model="phoneNumber" v-bind="phoneNumberProps"> </vue-tel-input> 
                    </b-input-group>
                    <div id="recaptcha-container" class="capcha"></div>
                    <b-button type="submit" class="btn btn-success mb-3" block v-on:click.prevent="login">
                      <span>Enviar codigo de verificación </span>
                    </b-button>
                  </div>
                  <div v-else>
                    <b-input-group>
                      <b-form-input type="text" class="form-control" 
                        placeholder="Codigo de confirmación" v-model="confirmationCode"/>
                    </b-input-group> 
                    <br/>
                    <b-button class="btn btn-success mb-3" block v-on:click.prevent="confirmCode">
                      <span>Iniciar sesión </span>
                    </b-button>
                  </div>
              </b-card-body>
            </b-card>
          </b-card-group>
        </b-col>
      </b-row>
    </div>
  </div>
</template>

<script>
import { VueTelInput } from 'vue-tel-input'
export default {
  name: 'Login', 
  data: function() {
      return {
          phoneNumber: '', 
          confirmationCode: '',
          confirmationCodeStage: false,
          confirmationResult: null, 
          error: false,
          redirect: null, 
          recaptcha: null, 
          errors: [], 
          phoneNumberProps:{
            mode: "international",
            defaultCountry: "CO",
            enabledCountryCode: true,
          }
      }
  }, 
  components: {
    VueTelInput,
  },
  created(){
    if (this.$route.query){
     let redirect =  this.$route.query.redirect 
     let id = this.$route.query.id 
     this.redirect = {'redirect': redirect, 'id': id}
    }
  },
  mounted(){
    // this.$firebase.auth().settings.appVerificationDisabledForTesting = true;
    this.recaptcha = new this.$firebase.auth.RecaptchaVerifier('recaptcha-container', {'size': 'normal'});
  },
  methods: {
      login: function(){
        this.errors = []
        this.$firebase.auth().languageCode = 'es';
        console.log(this.phoneNumber)
        this.$firebase.auth().signInWithPhoneNumber(this.phoneNumber, this.recaptcha).then((confirmationResult) => {
          this.confirmationResult = confirmationResult
          this.confirmationCodeStage = true 
        }).catch((error) =>  {
          console.log(error)
          this.errors = ['No has podido iniciar sesión']
          this.confirmationCodeStage = false 
          this.recaptcha.reset('recaptcha-container');
        });
      },
      confirmCode: function(){
        this.confirmationResult.confirm(this.confirmationCode).then((data) => {
          console.log('welcome :' + data)
          this.$router.push('/dashboard')
        }).catch((error) => {
          console.log(error)
          this.errors = ['Codigo de verificación invalido']
          this.recaptcha.reset('recaptcha-container');
          this.confirmationCodeStage = false 
        })
      }
  }
}
</script>

<style scoped>

    .sign-up-link {
        color: white;
    }

    .recaptcha-container{
      height: 100px;
      width: 100%;
    }
      
    #recaptcha-container {
      margin: auto; 
      padding: 1rem;
    }

</style>
