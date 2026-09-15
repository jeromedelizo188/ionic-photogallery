<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>My Photo Gallery</ion-title> </ion-toolbar>
    </ion-header>

    <ion-content class="ion-padding">
      <CameraComponent @photoCaptured="addPhoto" />
      <PhotoGalleryComponent :photos="photos" />
    </ion-content>
  </ion-page>
  </template>

<script setup lang="ts">
import {
  IonPage,
  IonHeader,
  IonToolbar,
  IonTitle,
  IonContent,
} from "@ionic/vue";
import CameraComponent from "@/components/CameraComponent.vue";
import PhotoGalleryComponent from "@/components/PhotoGalleryComponent.vue";
import { ref } from "vue";
const photos = ref<string[]>([]);
const addPhoto = (photo: string) => {
  photos.value.unshift(photo);
};

</script>

<style scoped>
.ion-padding {
  padding: 16px;
}  
.gallery-image {
  width: 100%;
  height: 180px;
}
.gallery-image::part(image) {
  object-fit: cover;
}
.empty-gallery {
  text-align: center;
  padding: 30px;
  color: #777;
}
</style>