<template>
  <div class="mx-auto d-flex align-canter justify-center" style="height: 98vh;">
    <v-card width = "500" rounded ="x1">
      <v-card-text class = "text-center my-8 mx-5">
        <v-icon size = "88" color="orange"> mdi-account-circle </v-icon>


        <p> Welcome back,please signin!</p>
        <v-form>
          <v-text-field prepend-inner-icon="mdi-account" variant="solo-filled" flat label="Username" rounded></v-text-field>
          <v-text-field prepend-inner-icon="mdi-lock" variant="solo-filled" flat label="Password" rounded></v-text-field>
          
          <v-btn color = "blue" class = "mt-2" rounded block>Signin</v-btn>

          <v-divider class = "my-8">OR</v-divider>
          <v-btn prepend-icon = "mdi-google" color="red" rounded block @click="loginWithGoogle">Signin with Google</v-btn>
        
        </v-form>

      </v-card-text>


    </v-card>

  </div>
</template>
 
<script setup lang="ts"> 
// @ts-nocheck 
const config = useRuntimeConfig() 
 
declare global { 
  interface Window { 
    google: any 
  } 
} 
 
const loginWithGoogle = () => { 
  const client = window.google.accounts.oauth2.initTokenClient({ 
    client_id: config.public.googleClientId, 
    scope: 'openid email profile', 
 
    callback: async (response: any) => { 
      const userInfo = await $fetch( 
        'https://www.googleapis.com/oauth2/v3/userinfo', 
        { 
          headers: { 
            Authorization: `Bearer ${response.access_token}` 
          } 
        } 
      ) 
 
      localStorage.setItem( 
        'google_user', 
        JSON.stringify(userInfo) 
      ) 
 
      localStorage.setItem( 
        'google_token', 
        response.access_token 
      ) 
 
      navigateTo('/') 
    } 
  }) 
 
  client.requestAccessToken();
} 
</script>