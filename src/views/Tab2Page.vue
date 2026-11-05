<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>Posts</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content :fullscreen="true">

      <ion-list>
        <ion-item v-for="post in posts" :key="post.id">
          <ion-label>
            <h2>{{ post.title }}</h2>
            <p>{{ post.body }}</p>
            <small>Usuario: {{ post.userId }}</small>
          </ion-label>
        </ion-item>
      </ion-list>

    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { IonPage, IonHeader, IonToolbar, IonTitle, IonContent, IonList, IonItem, IonLabel } from '@ionic/vue';
import { ref } from 'vue';
import { Posts } from '@/interfaces/posts';

const posts = ref<Posts[]>([]);

 async function cargarPosts() { /*AQUI LE AGREGUE ASYNC PARA QUE FUNCIONE EL AWAIT */

const response = await fetch('https://jsonplaceholder.typicode.com/posts');
const data = await response.json();
posts.value = data;
//El fetch sirve para hacer peticiones a una API, en este caso a
// la API de JSONPlaceholder que nos devuelve un array de objetos con información de posts. 
// Luego, usamos el método .json() para convertir la respuesta en un objeto JavaScript y lo asignamos a la variable posts.
// value para que se muestre en la vista.
//Pero use async y await para que espere a que se resuelva la promesa antes de asignar el valor a posts.value,
// y así evitar que se muestre un array vacío en la vista antes de que se carguen los datos de la API. 
// De esta manera, cuando se renderiza la vista,
// ya se tiene la información de los posts y se puede mostrar correctamente en la lista de la vista.
  /*fetch('https://jsonplaceholder.typicode.com/posts')
    .then(response => response.json())
    .then(json => {
      console.log(json);
      posts.value = json;
    });*/

}

cargarPosts();

</script>
