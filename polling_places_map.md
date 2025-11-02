---
layout: default
title: Polling Place Changes Map
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
    padding: 15px !important;
    border-radius: 4px !important;
    box-shadow: 0 1px 3px rgba(0,0,0,0.3) !important;
    line-height: 24px !important;
    color: #555 !important;
    font-size: 13px !important;
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
    margin-top: 2px !important;
    opacity: 0.8 !important;
    display: block !important;
    border: 1px solid #999 !important;
  }
  
  .info-box {
    background: white;
    padding: 15px;
    border-radius: 8px;
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
    margin-bottom: 20px;
  }
  
  .info-box h3 {
    margin-top: 0;
    color: #333;
  }
  
  .info-box p {
    margin: 5px 0;
  }
  
  .popup-content {
    font-size: 12px;
    line-height: 1.4;
  }
  
  .popup-content strong {
    color: #333;
  }
  
  .popup-changes {
    margin-top: 10px;
    padding-top: 10px;
    border-top: 1px solid #ddd;
  }
  
  .change-item {
    margin: 5px 0;
    padding-left: 15px;
  }
</style>

<div class="info-box">
  <h3>SEMS 2025 Special Election - Polling Place Changes</h3>
  <p>This map shows polling place changes (moved, closed, or address corrections) for the 2025 Special Election in Mississippi. Click on a county to see detailed information about changes in that area.</p>
  <p><strong>Total Changes:</strong> 25 across 9 counties</p>
</div>

<div class="map-container">
  <div id="polling-places-map" class="map"></div>
</div>

<script src="https://unpkg.com/leaflet/dist/leaflet.js"></script>
<script>
    document.addEventListener("DOMContentLoaded", function () {
      
      // Function to get color based on number of changes
      function getColor(changeCount) {
        if (changeCount === 0) return '#f0f0f0';
        if (changeCount === 1) return '#ffffcc';
        if (changeCount === 2) return '#ffeda0';
        if (changeCount === 3) return '#fed976';
        if (changeCount === 4) return '#feb24c';
        if (changeCount === 5) return '#fd8d3c';
        return '#e31a1c';
      }

      // Function to style counties
      function styleCounty(feature) {
        const changeCount = feature.properties.change_count || 0;
        const hasChanges = feature.properties.has_changes || false;
        
        return {
          fillColor: getColor(changeCount),
          weight: hasChanges ? 2 : 1,
          opacity: 1,
          color: hasChanges ? '#d32f2f' : '#999',
          fillOpacity: 0.8
        };
      }

      // Function to add legend
      function addLegend(map) {
        const legend = L.control({position: 'bottomright'});
  
        legend.onAdd = function (map) {
          const div = L.DomUtil.create('div', 'legend');
    
          div.innerHTML = '<h4>Polling Place Changes</h4>';
          
          const ranges = [
            {label: '0 changes', color: '#f0f0f0'},
            {label: '1 change', color: '#ffffcc'},
            {label: '2 changes', color: '#ffeda0'},
            {label: '3 changes', color: '#fed976'},
            {label: '4 changes', color: '#feb24c'},
            {label: '5 changes', color: '#fd8d3c'},
            {label: '6+ changes', color: '#e31a1c'}
          ];
          
          ranges.forEach(function(range) {
            const i = document.createElement('i');
            i.style.backgroundColor = range.color;
            const label = document.createElement('label');
            label.appendChild(i);
            label.appendChild(document.createTextNode(range.label));
            div.appendChild(label);
            div.appendChild(document.createElement('br'));
          });
          
          div.innerHTML += '<hr style="margin: 10px 0; border: none; border-top: 1px solid #ddd;">';
          div.innerHTML += '<p style="margin: 5px 0 0 0; font-size: 11px;"><strong>Red border:</strong> County has changes</p>';
    
          return div;
        };
  
        legend.addTo(map);
      }
      
      // Function to create popup content
      function createPopupContent(feature) {
        const props = feature.properties;
        const changeCount = props.change_count || 0;
        let content = `<div class="popup-content">`;
        
        content += `<strong>${props.NAME}</strong><br>`;
        
        if (changeCount > 0) {
          const types = props.change_types || {};
          content += `<div class="popup-changes">`;
          content += `<strong>${changeCount} Total Changes:</strong><br>`;
          
          if (types['Moved']) {
            content += `<span class="change-item">📍 Moved: ${types['Moved']}</span><br>`;
          }
          if (types['Closed']) {
            content += `<span class="change-item">🚫 Closed: ${types['Closed']}</span><br>`;
          }
          if (types['Address Issue']) {
            content += `<span class="change-item">⚠️ Address Issue: ${types['Address Issue']}</span><br>`;
          }
          content += `</div>`;
        } else {
          content += `<em>No polling place changes</em>`;
        }
        
        content += `</div>`;
        return content;
      }

      // Initialize map
      const pollingMap = L.map('polling-places-map', {
        scrollWheelZoom: false,
        zoomControl: true
      }).setView([32.3547, -89.3985], 7);
      
      L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
        attribution: '&copy; OpenStreetMap contributors',
        maxZoom: 18
      }).addTo(pollingMap);

      // Load counties GeoJSON
      fetch('assets/data/counties_with_changes.geojson')
        .then(res => res.json())
        .then(data => {
          const geoJsonLayer = L.geoJSON(data, {
            style: styleCounty,
            onEachFeature: function (feature, layer) {
              layer.bindPopup(createPopupContent(feature));
              
              // Highlight on hover
              layer.on('mouseover', function() {
                this.setStyle({
                  weight: 3,
                  opacity: 1
                });
              });
              
              layer.on('mouseout', function() {
                pollingMap.closePopup();
                const changeCount = feature.properties.change_count || 0;
                this.setStyle({
                  weight: changeCount > 0 ? 2 : 1,
                  opacity: 1
                });
              });
            }
          }).addTo(pollingMap);
          
          pollingMap.fitBounds(geoJsonLayer.getBounds());
          addLegend(pollingMap);
        })
        .catch(err => console.error('Error loading counties:', err));
    });
</script>

---

## About This Map

This map visualizes the polling place changes for Mississippi's 2025 Special Election. Changes include:

- **Moved**: Polling places that have relocated to a new facility
- **Closed**: Polling places that are no longer in operation
- **Address Issue**: Corrections to polling place addresses in SEMS

### Data Summary

- **Total Changes:** 25 precincts
- **Counties Affected:** 9

| County | Changes | Details |
|--------|---------|---------|
| Tallahatchie | 5 | 3 Moved, 2 Closed |
| Hinds | 6 | 5 Moved, 1 Address Issue |
| Desoto | 3 | 3 Moved |
| Forrest | 3 | 2 Address Issues, 1 Moved |
| Perry | 3 | 3 Address Issues |
| Madison | 2 | 1 Moved, 1 Closed |
| Bolivar | 1 | 1 Address Issue |
| Panola | 1 | 1 Closed |
| Tunica | 1 | 1 Address Issue |

The counties without changes are shown in light gray. Counties with changes are highlighted with red borders, with color intensity increasing by number of changes.
