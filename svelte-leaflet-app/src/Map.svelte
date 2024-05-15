<script>
  import { onMount } from 'svelte';
  import L from 'leaflet';
  import Papa from 'papaparse';
  import 'leaflet/dist/leaflet.css';
  import 'leaflet.markercluster';
  import 'leaflet.markercluster/dist/MarkerCluster.Default.css';
  import 'leaflet.markercluster/dist/MarkerCluster.css';


  // Set Leaflet icon paths correctly
  delete L.Icon.Default.prototype._getIconUrl;

  L.Icon.Default.mergeOptions({
    iconRetinaUrl: 'https://unpkg.com/leaflet@1.7.1/dist/images/marker-icon-2x.png',
    iconUrl: 'https://unpkg.com/leaflet@1.7.1/dist/images/marker-icon.png',
    shadowUrl: 'https://unpkg.com/leaflet@1.7.1/dist/images/marker-shadow.png',
  });

  let map;

  onMount(() => {
    map = L.map('map', {
      center: [50.1109, 8.6821], // Central location in Germany
      zoom: 6,
      scrollWheelZoom: true
    });

    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
      maxZoom: 19,
      attribution: '© OpenStreetMap contributors'
    }).addTo(map);

    // Load and parse the CSV data
    fetch('./data.csv')
      .then(response => response.text())
      .then(csvString => {
        const results = Papa.parse(csvString, { header: true, dynamicTyping: true });
        results.data.forEach(row => {
          L.marker([row.Latitude, row.Longitude])
            .bindPopup(`<strong>${row.CustomerName}</strong><br>${row.FullAddress}`)
            .addTo(map);
        });
      });
  });
</script>

<style>
  #map {
    position: absolute;
    top: 0;
    bottom: 0;
    left: 0;
    right: 0;
  }
</style>

<div id="map"></div>
