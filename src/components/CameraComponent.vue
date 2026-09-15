<template>
    <ion-card>
        <ion-card-header> <ion-card-title>Camera</ion-card-title> </ion-card-header>
        <ion-card-content> 
            <ion-button expand="block" @click="takePicture"> 
                <ion-icon slot ="start" :icon="cameraIcon" /> Take Picture
            </ion-button>
            <ion-text v-if="statusMessage">
                <p>{{ statusMessage }}</p>
            </ion-text>
            <ion-text v-if="errorMessage" color="danger">
                <p>{{ errorMessage }}</p>
            </ion-text>
        </ion-card-content>
    </ion-card>
</template>
<script setup lang ="ts"> 
import {
    IonButton,
    IonCard,
    IonCardContent,
    IonCardHeader,
    IonCardTitle,
    IonIcon,
    IonText,
} from "@ionic/vue";
import { camera as cameraIcon } from "ionicons/icons";
import { Camera } from "@capacitor/camera";
import { ref }  from "vue";
const errorMessage = ref("");
const statusMessage = ref("");
const emit = defineEmits<{ (event: "photoCaptured", photo: string): void }> ();
const takePicture = async () => {
    errorMessage.value = "";
    statusMessage.value = "";
    try {
        const photo = await Camera.takePhoto ({ quality: 90, saveToGallery: false });
        statusMessage.value = `Keys: ${Object.keys(photo).join(', ')} | thumb: ${photo.thumbnail ? 'yes (' + photo.thumbnail.length + ')' : 'no'} | webPath: ${photo.webPath ? 'yes' : 'no'}`;
        const mime = photo.thumbnail?.startsWith("/9j/") ? "image/jpeg" : "image/png";
        const imageSrc = photo.thumbnail
            ? `data:${mime};base64,${photo.thumbnail}`
            : photo.webPath;
        if (imageSrc) {
            emit("photoCaptured", imageSrc);
        } else {
            statusMessage.value += " | NO SOURCE TO DISPLAY";
        }
    } catch (error) {
      console.error(error);
      errorMessage.value = `Error: ${error instanceof Error ? error.message : String(error)}`;
    }
}
</script>