---
layout: default
title: Mississippi Polling Place Changes - 2025 Special Election
---

<link rel="stylesheet" href="https://unpkg.com/leaflet/dist/leaflet.css" />
<style>
  .map-container {
    margin: 0 auto 40px auto;
    max-width: 1200px;
    background: white;
    padding: 20px;
    border-radius: 8px;
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
  }
  .map {
    height: 600px;
    margin-top: 1em;
    border-radius: 4px;
  }
  .legend {
    background: white !important;
    padding: 10px !important;
    border-radius: 4px !important;
    box-shadow: 0 1px 3px rgba(0,0,0,0.3) !important;
    line-height: 24px !important;
    color: #555 !important;
  }
  .legend h4 {
    margin: 0 0 10px 0 !important;
    color: #333 !important;
    font-size: 14px !important;
  }
  .legend i {
    width: 18px !important;
    height: 18px !important;
    float: left !important;
    margin-right: 8px !important;
    opacity: 0.7 !important;
    display: block !important;
    border: 1px solid #999 !important;
  }
</style>

<div class="map-container">
  <h2>Mississippi Polling Place Changes Map</h2>
  <p>This map shows the 9 counties with polling place changes for the 2025 special election.</p>
  <div id="polling-map" class="map"></div>
</div>

<script src="https://unpkg.com/leaflet/dist/leaflet.js"></script>
<script>
document.addEventListener("DOMContentLoaded", function () {
  
  // Counties with polling place changes
  const changedCounties = [
    "Bolivar",
    "DeSoto",
    "Forrest",
    "Hinds",
    "Madison",
    "Panola",
    "Perry",
    "Tallahatchie",
    "Tunica"
  ];

  // Initialize map
  const pollingMap = L.map('polling-map', {
    scrollWheelZoom: false,
    zoomControl: true
  }).setView([32.8, -89.5], 7);

  // Add basemap
  L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
    attribution: '&copy; OpenStreetMap contributors'
  }).addTo(pollingMap);

  // Style function
  function styleCounty(feature) {
    const countyName = feature.properties.NAME;
    const hasChanges = changedCounties.includes(countyName);

    if (hasChanges) {
      return {
        fillColor: "#6c0175",
        weight: 2,
        opacity: 1,
        color: "white",
        fillOpacity: 0.7
      };
    } else {
      return {
        fillColor: "#cccccc",
        weight: 1,
        opacity: 0.4,
        color: "#999",
        fillOpacity: 0.15
      };
    }
  }

  // Popup function
  function onEachFeature(feature, layer) {
    if (feature.properties && feature.properties.NAME) {
      const countyName = feature.properties.NAME;
      const hasChanges = changedCounties.includes(countyName);

      let popupContent = `<strong>${countyName} County</strong><br>`;
      
      if (hasChanges) {
        popupContent += `<em>Has polling place changes</em>`;
      } else {
        popupContent += `No changes reported`;
      }

      layer.bindPopup(popupContent);
    }
  }

  // Add legend
  function addLegend(map) {
    const legend = L.control({position: 'bottomright'});

    legend.onAdd = function (map) {
      const div = L.DomUtil.create('div', 'legend');
      
      div.innerHTML = '<h4>Polling Place Changes</h4>';
      div.innerHTML += '<i style="background:#6c0175"></i> Counties with changes (9)<br>';
      div.innerHTML += '<i style="background:#cccccc; opacity: 0.3"></i> No changes (73)';
      
      return div;
    };

    legend.addTo(map);
  }

  // Load county GeoJSON
  fetch('/2025-elec-maps/assets/data/ms-counties.geojson')
    .then(response => response.json())
    .then(data => {
      const geoJsonLayer = L.geoJSON(data, {
        style: styleCounty,
        onEachFeature: onEachFeature
      }).addTo(pollingMap);

      pollingMap.fitBounds(geoJsonLayer.getBounds());
      addLegend(pollingMap);
    })
    .catch(error => {
      console.error('Error loading GeoJSON:', error);
      alert('Could not load county data. Check the file path and format.');
    });
});
</script>