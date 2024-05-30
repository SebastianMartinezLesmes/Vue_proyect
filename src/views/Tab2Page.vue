<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>Locations of Rick and Morty</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content :fullscreen="true">
      <div id="content_page">
        <div id="izquierda">
          <ion-accordion-group>
            <ion-accordion v-for="dimension in dimensions" :key="dimension.dimension">
              <ion-item slot="header">
                <ion-label>{{ dimension.dimension }}</ion-label>
              </ion-item>
              <ion-list slot="content">
                <ion-item v-for="location in dimension.locations" :key="location.id">
                  <ion-label>
                    <h2>{{ location.name }}</h2>
                    <p>Type: {{ location.type }}</p>
                    <p>Residents: {{ location.residents.length }}</p>
                  </ion-label>
                </ion-item>
              </ion-list>
            </ion-accordion>
          </ion-accordion-group>
        </div>
        <div id="derecha">

        </div>
      </div>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue';
import { 
  IonPage, 
  IonHeader, 
  IonToolbar, 
  IonTitle, 
  IonContent, 
  IonAccordion, 
  IonAccordionGroup, 
  IonLabel, 
  IonItem,
  IonList,
  IonCardHeader,
  IonCardTitle,
  IonCardContent 
} from '@ionic/vue';
import '../theme/tab2Page.css';

// Define interfaces for Location and Dimension
interface Location {
  id: number;
  name: string;
  type: string;
  residents: string[];
  dimension: string;
}

interface Dimension {
  dimension: string;
  locations: Location[];
}

const dimensions = ref<Dimension[]>([]);

async function fetchLocations() {
  try {
    let page = 1;
    let fetchMore = true;
    const allLocations: Location[] = [];

    while (fetchMore) {
      const response = await fetch(`https://rickandmortyapi.com/api/location?page=${page}`);
      const data = await response.json();
      
      const locations = data.results.map((loc: any): Location => ({
        id: loc.id,
        name: loc.name,
        type: loc.type,
        residents: loc.residents,
        dimension: loc.dimension 
      }));
      allLocations.push(...locations);
      
      if (data.info.next) {
        page++;
      } else {
        fetchMore = false;
      }
    }

    const dimensionsMap = new Map<string, Dimension>();

    allLocations.forEach(location => {
      const dimension = location.dimension;
      if (!dimensionsMap.has(dimension)) {
        dimensionsMap.set(dimension, {
          dimension: dimension,
          locations: []
        });
      }
      dimensionsMap.get(dimension)!.locations.push(location);
    });

    dimensions.value = Array.from(dimensionsMap.values());
    console.log(dimensions.value); // Log the dimensions to verify structure
  } catch (error) {
    console.error('Error fetching locations:', error);
  }
}

onMounted(() => {
  fetchLocations();
});
</script>
