<template>
    <div id="map"></div>
    <div id="controls">
      <label for="color">Color:</label>
      <input type="color" id="color" v-model="color" @input="updateColor" />
      <label for="opacity">Opacity:</label>
      <input type="range" id="opacity" min="0" max="1" step="0.1" v-model="opacity" @input="updateOpacity" />
      <button @click="exportGeometry">Export Geometry</button>
    </div>
  </template>
  
<script>
import { ref, onMounted } from 'vue'
import maplibregl from 'maplibre-gl'
import * as turf from '@turf/turf'
import 'maplibre-gl/dist/maplibre-gl.css'

const MAP_STYLE_URL =
  'https://api.maptiler.com/maps/streets/style.json?key=get_your_own_OpIi9ZULNHzrESv6T2vL'
const MAP_CENTER = [30.5, 50.5]
const MAP_ZOOM = 13

export default {
  setup() {
    const map = ref(null)
    const color = ref('#088')
    const opacity = ref(0.8)

    onMounted(() => {
      initializeMap()
    })

    function initializeMap() {
      map.value = new maplibregl.Map({
        container: 'map',
        style: MAP_STYLE_URL,
        center: MAP_CENTER,
        zoom: MAP_ZOOM
      })

      map.value.on('load', () => {
        addMarkerToMap()
        addPolygonToMap()
        addPointToMap()
        addLineToMap()
      })
    }

    function addMarkerToMap() {
      new maplibregl.Marker().setLngLat(MAP_CENTER).addTo(map.value)
    }

    function addPolygonToMap() {
      let polygon = turf.polygon([
        [
          [30.499318908327723, 50.50151942932925],
          [30.50387316388384, 50.50148086424835],
          [30.50880249875422, 50.50673020144259],
          [30.506982451477143, 50.50866237989095],
          [30.50601434462334, 50.51167902474069],
          [30.499505403862457, 50.511833804916535],
          [30.499318908327723, 50.50151942932925]
        ]
      ])

      map.value.addSource('polygon', {
        type: 'geojson',
        data: polygon
      })

      map.value.addLayer({
        id: 'polygon',
        type: 'fill',
        source: 'polygon',
        layout: {},
        paint: {
          'fill-color': '#088',
          'fill-opacity': 0.8
        }
      })
    }

    function addPointToMap() {
      let point = turf.point([30.493996353506873, 50.512318021589834])
      map.value.addSource('point', {
        type: 'geojson',
        data: point
      })
      map.value.addLayer({
        id: 'point',
        type: 'circle',
        source: 'point',
        paint: {
          'circle-radius': 10,
          'circle-color': '#000'
        }
      })
    }

    function addLineToMap() {
      let line = turf.lineString([
        [30.48898370424007, 50.5071655201462],
        [30.49359633640637, 50.51180537610722]
      ])
      map.value.addSource('line', {
        type: 'geojson',
        data: line
      })
      map.value.addLayer({
        id: 'line',
        type: 'line',
        source: 'line',
        paint: {
          'line-color': '#888',
          'line-width': 8
        }
      })
    }

    function updateColor() {
      map.value.setPaintProperty('polygon', 'fill-color', color.value)
      map.value.setPaintProperty('line', 'line-color', color.value)
      map.value.setPaintProperty('point', 'circle-color', color.value)
    }

 function updateOpacity() {
  map.value.setPaintProperty('polygon', 'fill-opacity', opacity.value)
  map.value.setPaintProperty('line', 'line-opacity', opacity.value)
  map.value.setPaintProperty('point', 'circle-opacity', opacity.value)
}


    function exportGeometry() {
      const polygonData = map.value.getSource('polygon')._data
      const lineData = map.value.getSource('line')._data
      const pointData = map.value.getSource('point')._data

      const combinedData = {
        type: 'FeatureCollection',
        features: [polygonData.features[0], lineData.features[0], pointData.features[0]]
      }

      const blob = new Blob([JSON.stringify(combinedData)], {
        type: 'application/json;charset=utf-8'
      })
      console.log(combinedData)
      let link = document.createElement('a')
      link.href = URL.createObjectURL(blob)
      link.download = 'geometry.geojson'
      link.click()
    }

    return {
      exportGeometry,
      color,
      opacity,
      updateColor,
      updateOpacity
    }
  }
}
</script>

<style scoped>
#map {
  height: 90vh; /* Adjust as needed */
}

#controls {
  position: absolute;
  top: 10px;
  left: 10px;
  z-index: 1;
  background: white;
  padding: 10px;
  border-radius: 3px;
}
</style>
