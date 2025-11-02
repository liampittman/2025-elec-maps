---
layout: default
title: Mississippi Counties Choropleth (Total)
---

<link rel="stylesheet" href="https://unpkg.com/leaflet/dist/leaflet.css" />

<style>
.map-container { margin: 0 auto 40px auto; max-width: 1200px; background: white; padding: 20px; border-radius: 8px; box-shadow: 0 2px 4px rgba(0,0,0,0.1); }
.map { height: 600px; margin-top: 1em; border-radius: 4px; }
.legend { background: white; padding: 10px; border-radius: 4px; box-shadow: 0 1px 3px rgba(0,0,0,0.3); line-height: 24px; color: #555; }
</style>

<div class="map-container">
  <div id="county-map" class="map"></div>
</div>

<script src="https://unpkg.com/leaflet/dist/leaflet.js"></script>
<script>
document.addEventListener("DOMContentLoaded", function() {
    // Set realistic min and max after inspecting your data.
    const MIN = 0;
    const MAX = 10000; // Replace with your dataset's actual max

    function getColor(value) {
        const clampedValue = Math.max(MIN, Math.min(MAX, value));
        const normalized = (clampedValue - MIN) / (MAX - MIN);
        // Example: gradient from #fffcd4 to #6c0175 (light yellow to purple)
        const r = Math.round(255 - (255 - 108) * normalized);
        const g = Math.round(252 - (252 - 1) * normalized);
        const b = Math.round(212 - (212 - 117) * normalized);
        return `rgb(${r},${g},${b})`;
    }

    function styleCounty(feature) {
        const total = feature.properties.total;
        return {
            fillColor: getColor(total),
            weight: 1,
            opacity: 1,
            color: '#999',
            fillOpacity: 0.8,
        };
    }

    function onEachFeature(feature, layer) {
        const props = feature.properties;
        let popup = `<strong>${props.county_name || props.NAME}</strong>`;
        if (typeof props.total !== "undefined") {
            popup += `<br>Total: <strong>${props.total}</strong>`;
        }
        layer.bindPopup(popup);
    }

    const countyMap = L.map('county-map', { scrollWheelZoom: false }).setView([32.3547, -89.3985], 7);
    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
        attribution: '© OpenStreetMap contributors'
    }).addTo(countyMap);

    fetch('assets/data/ms-counties.geojson')
        .then(response => response.json())
        .then(data => {
            const geoJsonLayer = L.geoJSON(data, {
                style: styleCounty,
                onEachFeature: onEachFeature
            }).addTo(countyMap);
            countyMap.fitBounds(geoJsonLayer.getBounds());

            // Add the legend
            const legend = L.control({position: 'bottomright'});
            legend.onAdd = function (map) {
                let div = L.DomUtil.create('div', 'legend');
                div.innerHTML = '<h4>Total by County</h4>';
                div.innerHTML += `<i style="background:${getColor(MIN)}"></i> ${MIN}<br>`;
                div.innerHTML += `<i style="background:${getColor(MAX)}"></i> ${MAX}<br>`;
                return div;
            };
            legend.addTo(countyMap);
        })
        .catch(err => console.error("Error loading counties:", err));
});
</script>