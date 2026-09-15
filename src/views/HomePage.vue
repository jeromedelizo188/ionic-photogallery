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
import { ref, onMounted } from "vue";
import { Filesystem, Directory, Encoding } from "@capacitor/filesystem";
import { Preferences } from "@capacitor/preferences";

const photos = ref<string[]>([]);
const PHOTO_DIR = "photos";
const PREFS_KEY = "photoFiles";

const blobToBase64 = (blob: Blob): Promise<string> =>
  new Promise((resolve, reject) => {
    const reader = new FileReader();
    reader.onloadend = () => resolve((reader.result as string).split(",")[1]);
    reader.onerror = reject;
    reader.readAsDataURL(blob);
  });

const loadPhoto = async (filename: string) => {
  try {
    const file = await Filesystem.readFile({
      path: `${PHOTO_DIR}/${filename}`,
      directory: Directory.Data,
      encoding: Encoding.UTF8,
    });
    photos.value.push(`data:image/jpeg;base64,${file.data}`);
  } catch (error) {
    console.error("Failed to load photo:", filename, error);
  }
};

const loadPhotos = async () => {
  try {
    const result = await Preferences.get({ key: PREFS_KEY });
    const filenames: string[] = result.value ? JSON.parse(result.value) : [];
    for (const filename of filenames) {
      await loadPhoto(filename);
    }
  } catch (error) {
    console.error("Failed to load photo list:", error);
  }
};

const addPhoto = async (photo: string) => {
  try {
    const response = await fetch(photo);
    const blob = await response.blob();
    const base64 = await blobToBase64(blob);
    const filename = `${Date.now()}.jpg`;

    await Filesystem.writeFile({
      path: `${PHOTO_DIR}/${filename}`,
      data: base64,
      directory: Directory.Data,
      encoding: Encoding.UTF8,
    });

    photos.value.unshift(`data:image/jpeg;base64,${base64}`);

    const stored = await Preferences.get({ key: PREFS_KEY });
    const filenames: string[] = stored.value ? JSON.parse(stored.value) : [];
    filenames.unshift(filename);
    await Preferences.set({ key: PREFS_KEY, value: JSON.stringify(filenames) });
  } catch (error) {
    console.error("Failed to save photo:", error);
  }
};

onMounted(() => {
  loadPhotos();
});

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
</style>