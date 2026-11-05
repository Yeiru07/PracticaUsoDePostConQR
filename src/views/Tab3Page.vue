<script setup lang="ts">

import { ref, onMounted } from 'vue';

import {
  IonPage,
  IonHeader,
  IonToolbar,
  IonTitle,
  IonContent,
  IonItem,
  IonLabel,
  IonInput,
  IonTextarea,
  IonButton,
  IonSelect,
  IonSelectOption,
  IonText
} from '@ionic/vue';

import { CapacitorBarcodeScanner } from '@capacitor/barcode-scanner';

// Opciones del escáner
const options = {
  hint: 0,
  scanInstructions: 'Escanee un código QR',
  cameraDirection: 1
};

// Función para utilizar el escáner
const scanBarcode = (options: any) => {
  return CapacitorBarcodeScanner.scanBarcode(options);
};

// Usuarios
const usuarios = ref<any[]>([]);

// Formulario
const form = ref({
  name: '',
  body: '',
  userId: 0
});

// Mensajes
const mensaje = ref('');

// Obtener usuarios
const obtenerUsuarios = async () => {

  try {

    const response = await fetch(
      'https://jsonplaceholder.typicode.com/users'
    );

    if (!response.ok) {
      throw new Error('No se pudieron obtener los usuarios');
    }

    usuarios.value = await response.json();

  } catch (error) {

    console.error('Error al obtener usuarios:', error);

    mensaje.value = 'Error al cargar los usuarios';

  }

};


const escanearQR = async () => {

  try {

    mensaje.value = '';

    const data = await scanBarcode(options);

    console.log('Resultado completo:', data);

    const contenido = data.ScanResult;

    console.log('Contenido QR:', contenido);

    if (!contenido) {
      mensaje.value = 'El código QR está vacío';
      return;
    }

    // Separar las líneas del QR
    const lineas = contenido.split(/\r?\n/);

    const campos: Record<string, string> = {};

    lineas.forEach((linea: string) => {

      const separador = linea.indexOf(':');

      if (separador !== -1) {

        const clave = linea.substring(0, separador)
          .trim()
          .toLowerCase();

        const valor = linea.substring(separador + 1).trim();

        campos[clave] = valor;

      }

    });

    // Aceptar diferentes nombres para los campos
    const name = campos['name'] || campos['title'] || '';
    const body = campos['body'] || '';
    const userId = Number(campos['userid'] || campos['user']);

    // Validar contenido
    if (!name || !body || !Number.isInteger(userId) || userId <= 0) {

      mensaje.value = 'El formato del código QR no es válido';

      console.error('Contenido QR incorrecto:', contenido);

      return;

    }

    // Rellenar formulario
    form.value.name = name;
    form.value.body = body;
    form.value.userId = userId;

    mensaje.value = 'Código QR leído correctamente';

    console.log('Formulario actualizado:', form.value);

  } catch (error) {

    console.error('Error al escanear QR:', error);

    mensaje.value = 'No se pudo escanear el código QR';

  }

};


// Enviar formulario
const crearPost = async () => {

  try {

    mensaje.value = '';

    if (
      !form.value.name.trim() ||
      !form.value.body.trim() ||
      !form.value.userId
    ) {

      mensaje.value = 'Complete todos los campos';

      return;

    }

    const response = await fetch(
      'https://jsonplaceholder.typicode.com/posts',
      {
        method: 'POST',

        body: JSON.stringify({
          title: form.value.name,
          body: form.value.body,
          userId: form.value.userId
        }),

        headers: {
          'Content-type': 'application/json; charset=UTF-8'
        }
      }
    );

    if (!response.ok) {
      throw new Error('Error al crear la publicación');
    }

    const json = await response.json();

    console.log('Publicación creada:', json);

    mensaje.value = 'Publicación creada correctamente';

  } catch (error) {

    console.error('Error al enviar:', error);

    mensaje.value = 'Error al enviar el formulario';

  }

};

// Ejecutar al cargar
onMounted(() => {

  obtenerUsuarios();

});

</script>

<template>

  <IonPage>

    <IonHeader>

      <IonToolbar>

        <IonTitle>Crear publicación</IonTitle>

      </IonToolbar>

    </IonHeader>

    <IonContent class="ion-padding">

      <IonItem>

        <IonLabel position="stacked">Nombre</IonLabel>

        <IonInput v-model="form.name" placeholder="Ingrese el nombre" />

      </IonItem>

      <IonItem>

        <IonLabel position="stacked">Contenido</IonLabel>

        <IonTextarea v-model="form.body" placeholder="Ingrese el contenido" />

      </IonItem>

      <IonItem>

        <IonLabel position="stacked">Usuario</IonLabel>

        <IonSelect v-model="form.userId" placeholder="Seleccione un usuario">

          <IonSelectOption v-for="usuario in usuarios" :key="usuario.id" :value="usuario.id">
            {{ usuario.name }}
          </IonSelectOption>

        </IonSelect>

      </IonItem>

      <div class="ion-padding-top">

        <IonButton expand="block" @click="escanearQR">
          Escanear código QR
        </IonButton>

        <IonButton expand="block" @click="crearPost">
          Crear publicación
        </IonButton>

      </div>

      <IonText color="primary" v-if="mensaje">

        <p>{{ mensaje }}</p>

      </IonText>

    </IonContent>

  </IonPage>

</template>
