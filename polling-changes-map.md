---
layout: default
title: Mississippi Polling Place Changes - 2025 Special Election
---

<div id="ms-map" style="width: 100%; height: 600px;"></div>

<link rel="stylesheet" href="https://unpkg.com/leaflet/dist/leaflet.css" />
<script src="https://unpkg.com/leaflet/dist/leaflet.js"></script>

<script>
// Counties with polling place changes (from your CSV data)
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

// Initialize the map centered on Mississippi
var map = L.map('ms-map').setView([32.8, -89.5], 7);

// Add OpenStreetMap tiles as the basemap
L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
    maxZoom: 18,
    attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors'
}).addTo(map);

// Style function: highlight changed counties, grey-out others
function style(feature) {
    const countyName = feature.properties.NAME;
    
    if (changedCounties.includes(countyName)) {
        // Counties with changes: purple highlight
        return {
            fillColor: "#6c0175",
            weight: 2,
            opacity: 1,
            color: "#333",
            fillOpacity: 0.7
        };
    } else {
        // All other counties: translucent grey
        return {
            fillColor: "#cccccc",
            weight: 1,
            opacity: 0.5,
            color: "#999",
            fillOpacity: 0.25
        };
    }
}

// Add popup on click to show county name
function onEachFeature(feature, layer) {
    if (feature.properties && feature.properties.NAME) {
        const countyName = feature.properties.NAME;
        const hasChanges = changedCounties.includes(countyName);
        
        const popupText = hasChanges 
            ? `<strong>${countyName} County</strong><br>Has polling place changes`
            : `<strong>${countyName} County</strong><br>No changes reported`;
        
        layer.bindPopup(popupText);
    }
}

// Load your county GeoJSON and add to map
fetch('./assets/data/ms-counties.geojson')
    .then(response => response.json())
    .then(data => {    
        L.geoJSON(data, {
            style: style,
            onEachFeature: onEachFeature
        }).addTo(map);
    })
    .catch(error => {
        console.error('Error loading GeoJSON:', error);
        alert('Could not load county data. Check the file path and format.');
    });
</script>