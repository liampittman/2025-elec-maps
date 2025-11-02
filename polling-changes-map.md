---
layout: default
title: Mississippi County Changes Map
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
  <div id="county-map" class="map"></div>
</div>

<script src="https://unpkg.com/leaflet/dist/leaflet.js"></script>
<script>
document.addEventListener("DOMContentLoaded", function() {
    fetch('./assets/data/ms-counties.geojson')
      .then(res => res.json())
      .then(data => {
        // Get all actual 'total' values as numbers, dropping NaN/nulls
        const totals = data.features.map(f =>
            parseFloat(f.properties.total)
        ).filter(v => !isNaN(v));
        const min = Math.min(...totals);
        const max = Math.max(...totals);

        function getColor(value) {
            const v = Math.max(min, Math.min(max, value));
            const t = (v-min)/(max-min);
            // Linear interpolation, adjust colors as needed
            const r = Math.round(242 + (108-242)*t);
            const g = Math.round(231 + (1-231)*t);
            const b = Math.round(250 + (117-250)*t);
            return `rgb(${r},${g},${b})`;
        }

        function countyStyle(feature) {
            const t = parseFloat(feature.properties.total);
            return {
                fillColor: getColor(isNaN(t) ? min : t),
                weight: 1,
                opacity: 1,
                color: "#333",
                fillOpacity: 0.75,
            };
        }

        function onEachCounty(feature, layer) {
            const name = feature.properties.CONAME || "Unknown";
            const t = feature.properties.total == null || isNaN(parseFloat(feature.properties.total))
                ? "No data" : feature.properties.total;
            const popup = `<strong>${name} County</strong><br>Change: ${t}`;
            layer.bindPopup(popup);
        }

        var map = L.map('county-map', {
            scrollWheelZoom: false,
            zoomControl: true
        }).setView([32.3547, -89.3985], 7);

        L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
            attribution: '&copy; OpenStreetMap contributors'
        }).addTo(map);

        var counties = L.geoJSON(data, {
            style: countyStyle,
            onEachFeature: onEachCounty
        }).addTo(map);
        map.fitBounds(counties.getBounds());
        addLegend(map, min, max);

        function addLegend(map, min, max) {
            var legend = L.control({position: 'bottomright'});
            legend.onAdd = function (map) {
                var div = L.DomUtil.create('div', 'legend');
                div.innerHTML = `<h4>Change by County</h4>
                  <div style="background: linear-gradient(to right, #f2e7fa, #c292e7, #a839c6, #6c0175); height: 20px; width: 200px; border:1px solid #999; margin-bottom:5px"></div>
                  <div style="display: flex; justify-content: space-between; width: 200px; font-size:11px"><span>${min}</span><span>${max}</span></div>`;
                return div;
            };
            legend.addTo(map);
        }
    });
});
</script>
