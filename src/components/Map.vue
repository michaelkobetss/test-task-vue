<template>
  <div class="container mx-auto mt-4">
    <div class="flex">
      <div id="controls" class="controls">
        <label for="color">Color:</label>
        <input type="color" id="color" v-model="color" @change="updateColor" />
        <label for="opacity">Opacity:</label>
        <input
          type="range"
          id="opacity"
          min="0"
          max="1"
          step="0.1"
          v-model="opacity"
          @change="updateOpacity"
        />
        <button
          @click="exportGeometry"
          class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded"
        >
          Export Geometry
        </button>
        <button id="drawPolygon">Draw Polygon</button>
  <button id="drawLineString">Draw LineString</button>
  <button id="drawPoint">Draw Point</button>
  <button id="delete">Delete</button>
      </div>
      <div id="map" class="map"></div>
    </div>
    
  </div>
</template>

<script>
import { ref, onMounted } from 'vue'
import maplibregl from 'maplibre-gl'

import MapboxDraw from "@mapbox/mapbox-gl-draw";
import '@mapbox/mapbox-gl-draw/dist/mapbox-gl-draw.css'

import * as turf from '@turf/turf'
import 'maplibre-gl/dist/maplibre-gl.css'

const MAP_STYLE_URL =
  'https://api.maptiler.com/maps/streets/style.json?key=get_your_own_OpIi9ZULNHzrESv6T2vL'
const MAP_CENTER = [30.5, 50.5]
const MAP_ZOOM = 13

const polygon = turf.polygon([
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
const point = turf.point([30.493996353506873, 50.512318021589834])
const line = turf.lineString([
  [30.48898370424007, 50.5071655201462],
  [30.49359633640637, 50.51180537610722]
])

export default {
  setup() {
    const map = ref(null)
    const color = ref('#088000')
    let opacity = ref(0.8)
    let draw = null

    onMounted(() => {
      initializeMap()
      document.getElementById('drawPolygon').addEventListener('click', function() {
    draw.changeMode('draw_polygon');
  });
  document.getElementById('drawLineString').addEventListener('click', function() {
    draw.changeMode('draw_line_string');
  });
  document.getElementById('drawPoint').addEventListener('click', function() {
    draw.changeMode('draw_point');
  });
  document.getElementById('delete').addEventListener('click', function() {
    draw.changeMode('delete');
  });
    })

    function initializeMap() {
      map.value = new maplibregl.Map({
        container: 'map',
        style: MAP_STYLE_URL,
        center: MAP_CENTER,
        zoom: MAP_ZOOM
      })

      draw = new MapboxDraw({
        displayControlsDefault: false,
        controls: {
          polygon: true,
          trash: true
        }
      })
      
      
      map.value.on('load', () => {
        addPolygonToMap(polygon)
        addPointToMap(point)
        addLineToMap(line)
      })
      map.value.addControl(draw, 'top-right')
    }

    function addPolygonToMap(polygon) {
      map.value.addSource('polygon', {
        type: 'geojson',
        data: polygon
      })

      map.value.addLayer({
        id: 'polygon',
        type: 'fill',
        source: 'polygon',
        paint: {
          'fill-opacity': opacity.value,
          'fill-color': color.value
        }
      })
    }

    function addPointToMap(point) {
      map.value.addSource('point', {
        type: 'geojson',
        data: point
      })

      map.value.addLayer({
        id: 'point',
        type: 'circle',
        source: 'point',
        paint: {
          'circle-radius': 4,
          'circle-opacity': opacity.value,
          'circle-color': color.value
        }
      })
    }

    function addLineToMap(line) {
      map.value.addSource('line', {
        type: 'geojson',
        data: line
      })

      map.value.addLayer({
        id: 'line',
        type: 'line',
        source: 'line',
        paint: {
          'line-width': 4,
          'line-opacity': opacity.value,
          'line-color': color.value
        }
      })
    }

    function updateColor() {
      map.value.setPaintProperty('polygon', 'fill-color', color.value)
      map.value.setPaintProperty('line', 'line-color', color.value)
      map.value.setPaintProperty('point', 'circle-color', color.value)
    }

    function updateOpacity() {
      let numericOpacity = Number(opacity.value)
      map.value.setPaintProperty('polygon', 'fill-opacity', numericOpacity)
      map.value.setPaintProperty('line', 'line-opacity', numericOpacity)
      map.value.setPaintProperty('point', 'circle-opacity', numericOpacity)
    }

    function exportGeometry() {
      if (
        !map.value.getSource('polygon') ||
        !map.value.getSource('point') ||
        !map.value.getSource('line')
      ) {
        console.log('Map sources are not ready yet.')
        return
      }

      const drawnData = draw.getAll()

      const combinedData = {
        type: 'FeatureCollection',
        features: [...drawnData.features]
      }

      const blob = new Blob([JSON.stringify(combinedData)], {
        type: 'application/json;charset=utf-8'
      })
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
  /* position: absolute; */
  top: 10px;
  left: 10px;
  z-index: 1;
  background: var(--vt-c-black-soft);
  padding: 10px;

  display: flex;
}
#controls label,
#controls input {
  display: block;
  margin-bottom: 10px; /* Adjust as needed */
}

.mapboxgl-ctrl-top-right {
  z-index: 9999 !important;
}
</style>
