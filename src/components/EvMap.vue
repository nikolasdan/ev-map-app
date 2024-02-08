<script setup>
import { ref, onMounted } from 'vue';
import axios from 'axios';
import gmapTheme from './gmapTheme.json';

const center = ref({ lat: 47.14769614452696, lng: 27.58352623143465 });

const markers = ref([]);

const mapOptions = {
  styles: gmapTheme,
  streetViewControl: false,
  mapTypeControl: false
}

const modalData = ref({});

const modalOpen = ref(false);

const openModal = (marker) => {
  modalData.value = marker;
  modalOpen.value = true;
};

const closeModal = () => {
  modalOpen.value = false;
};

onMounted(async () => {
  try {
    const response = await axios.get('https://stg-be.gocharge.tech:8443/api/ev-chargers-data');
    const data = response.data.data;

    const parsedMarkers = data.map(item => {
      let marker = {
        position: { lat: parseFloat(item.ev_location.lat), lng: parseFloat(item.ev_location.lng) },
        info: item.ev_location.name,
        full_address: item.ev_location.full_address,
        max_power: item.max_power,
        price_per_kw: item.price_per_kw,
        currency: item.currency,
        status: item.status ? "Available" : "Not available",
        charger_type: item.chargers_guns[0].type,
        power: item.chargers_guns[0].kw_power,
        unit: item.chargers_guns[0].unit,
        is_fast: item.chargers_guns[0].is_fast ? "Fast" : "Slow",
      };

      if (item.chargers_guns[1] !== undefined) {
        marker.charger_type_two = item.chargers_guns[1].type;
        marker.power_two = item.chargers_guns[1].kw_power;
        marker.unit_two = item.chargers_guns[1].unit;
        marker.is_fast_two = item.chargers_guns[1].is_fast ? "Fast" : "Slow";
      }

      return marker;

    });

    markers.value = parsedMarkers;
  } catch (error) {
    console.error('Error fetching data:', error);
  }
});

</script>

<template>
  <GMapMap :center="center" :zoom="7"
    style="height : 100%; width : 100%; top : 0; left : 0; position : absolute; z-index : ;" :options="mapOptions">
    <GMapMarker v-for="(marker, index) in markers" :key="index" :position="marker.position"
      :icon="'https://app.gocharge.tech/icons/gmap_pin_available.png'" @click="openModal(marker)">
    </GMapMarker>
  </GMapMap>
  <div class="modal" v-if="modalOpen">
    <div class="modal-content">
      <span class="close" @click="closeModal">&times;</span>
      <h2 class="modal-info">{{ modalData.info }}</h2>
      <p class="modal-address">Address: {{ modalData.full_address }}</p>
      <p class="modal-max-power">Max Power: {{ modalData.max_power + " " + modalData.unit }}</p>
      <p class="modal-price">Price per KW: {{ modalData.price_per_kw + " " + modalData.currency }}</p>
      <p class="modal-status">Status: {{ modalData.status }}</p>

      <p class="modal-charger-type">Type: {{ modalData.charger_type }}</p>
      <p class="modal-power">Power: {{ modalData.power + " " + modalData.unit }} {{ modalData.is_fast }}</p>
      <p v-if="modalData.charger_type_two" class="modal-charger-two">Type: {{ modalData.charger_type_two }}</p>
      <p v-if="modalData.power_two && modalData.unit_two && modalData.is_fast_two" class="modal-power-two">Power: {{
        modalData.power_two + " " + modalData.unit }} {{ modalData.is_fast_two }}</p>
    </div>
  </div>
</template>

<style scoped>
.modal {
  display: block;
  position: fixed;
  z-index: 1;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  overflow: auto;
  background-color: rgba(0, 0, 0, 0.4);
  backdrop-filter: blur(2px);
}

h2 {
  text-align: center;
  font-family: 'Inter', sans-serif;
}

p {
  font-family: 'Inter', sans-serif;
  font-weight: 400;
  text-align: center;
}

.modal-content {
  background-color: #333333a6;
  backdrop-filter: blur(5px);
  color: #ffffff;
  margin: 15% auto;
  padding: 20px;
  border: 1px solid #888;
  width: 40%;
  border-radius: 13px;
}

@media (max-width: 768px) {
  .modal-content {
    width: 80%;
  }
}

.close {
  color: #aaa;
  float: right;
  font-size: 28px;
  font-weight: bold;
}

.close:hover,
.close:focus {
  color: rgb(255, 255, 255);
  text-decoration: none;
  cursor: pointer;
}
</style>
