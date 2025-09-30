<template>
  <ion-page>
    <ion-header class="header">
      <ion-toolbar>
          <ion-title class="text-center w-full">GENIUS SUDOKU</ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-content :fullscreen="true" class="ion-justify-content-center">
      <SudokuGame />
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { IonPage, IonHeader, IonToolbar, IonTitle, IonContent } from '@ionic/vue';
import SudokuGame from '@/components/SudokuGame.vue';
</script>
