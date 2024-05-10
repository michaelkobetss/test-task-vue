<template>
    <div class="app-root">
      <div class="map-container">
        <div id="map"></div>
      </div>
    </div>
  </template>
  
  <script>
  import { onMounted } from 'vue'
  import maplibregl from 'maplibre-gl'
  import * as turf from '@turf/turf'
  
  export default {
    setup() {
      onMounted(() => {
        const map = new maplibregl.Map({
          container: 'map',
          style: 'https://demotiles.maplibre.org/style.json',
          center: [0, 0],
          zoom: 1
        })
  
        // Додайте вашу геометрію тут
        // Наприклад, для додавання маркера:
        new maplibregl.Marker().setLngLat([0, 0]).addTo(map)
  
        // Додавання полігона
        const polygon = turf.polygon([[
          [-5, 5],
          [5, -5],
          [-5, -5],
          [-5, 5]
        ]])
        map.addLayer({
          'id': 'polygon',
          'type': 'fill',
          'source': {
            'type': 'geojson',
            'data': polygon
          },
          'layout': {},
          'paint': {
            'fill-color': '#088',
            'fill-opacity': 0.8
          }
        })
  
        // Додавання лінії
        const line = turf.lineString([[-10, 10], [10, -10]])
        map.addLayer({
          'id': 'line',
          'type': 'line',
          'source': {
            'type': 'geojson',
            'data': line
          },
          'layout': {},
          'paint': {
            'line-color': '#f00',
            'line-width': 2
          }
        })
  
        // Додавання точки за допомогою turf
        const point = turf.point([15, 15])
        new maplibregl.Marker().setLngLat(point.geometry.coordinates).addTo(map)
      })
    }
  }
  </script>
  
  
  <style scoped>

  .map-container{
    margin: auto;
    display: flex;
    justify-content: center;
  }
  
  #map {
    width: 40rem;
    height: 30rem;
    border: 1px solid #ccc;
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.5);
    overflow: hidden;
    position: relative;
  }
  </style>
  