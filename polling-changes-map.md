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
  .legend-item {
    margin-bottom: 5px !important;
    clear: both !important;
  }
</style>

<div class="map-container">
  <div id="polling-map" class="map"></div>
</div>

<script src="https://unpkg.com/leaflet/dist/leaflet.js"></script>
<script>
    document.addEventListener("DOMContentLoaded", function () {
      
      // County data with issues
      const countyIssues = {
        'Madison': { primary: 'CLOSED', all: ['MOVED', 'CLOSED'] },
        'Panola': { primary: 'CLOSED', all: ['CLOSED'] },
        'Tallahatchie': { primary: 'CLOSED', all: ['MOVED', 'CLOSED'] },
        'DeSoto': { primary: 'MOVED', all: ['MOVED'] },
        'Forrest': { primary: 'MOVED', all: ['MOVED', 'ADDRESS_ISSUES'] },
        'Hinds': { primary: 'MOVED', all: ['MOVED', 'ADDRESS_ISSUES'] },
        'Bolivar': { primary: 'ADDRESS_ISSUES', all: ['ADDRESS_ISSUES'] },
        'Perry': { primary: 'ADDRESS_ISSUES', all: ['ADDRESS_ISSUES'] },
        'Tunica': { primary: 'ADDRESS_ISSUES', all: ['ADDRESS_ISSUES'] }
      };

      // Precinct details by county
      const countyDetails = {
        'Madison': {
          closed: ['RIDGECREST BAPTIST CHURCH (314)'],
          moved: ['Liberty M. B. Church (406)']
        },
        'Panola': {
          closed: ['PATTON LANE COMMUNITY CENTER (526)']
        },
        'Tallahatchie': {
          closed: ['BRAZIL (202)', 'LEVERETTE (406)'],
          moved: ['SUMNER BEAT #2 (203)', 'GLENDORA (403)', 'SUMNER BEAT #5 (502)']
        },
        'DeSoto': {
          moved: ['Rejoice Church (201)', 'Pathway Church (202)', 'St Paul Missionary Baptist Church (205)']
        },
        'Forrest': {
          moved: ['D.R.E.A.M. Building (CRT)'],
          address: ['HARDY STREET* (HS)', 'WESTSIDE* (WS)']
        },
        'Hinds': {
          moved: ['PRECINCT 38 (38)', 'PRECINCT 57 (57)', 'PRECINCT 59 (59)', 'PRECINCT 61 (61)', 'PRECINCT 63 (63)'],
          address: ['PRECINCT 70 (70)']
        },
        'Bolivar': {
          address: ['SHELBY (SB)']
        },
        'Perry': {
          address: ['HINTONVILLE 4090* (4090)', 'BEAUMONT PRECINCT 5120* (5120)', 'THOMPSON HILL 5130* (5130)']
        },
        'Tunica': {
          address: ['MHOON LANDING* (5)']
        }
      };

      // Function to get color based on issue type
      function getColor(primary) {
        switch(primary) {
          case 'CLOSED':
            return '#d32f2f'; // Dark red
          case 'MOVED':
            return '#f57c00'; // Orange
          case 'ADDRESS_ISSUES':
            return '#fbc02d'; // Yellow
          default:
            return '#e0e0e0'; // Light gray for no issues
        }
      }

      // Function to style counties
      function styleCounty(feature) {
        const countyName = feature.properties.name || feature.properties.NAME || feature.properties.Name;
        const issueData = countyIssues[countyName];
        
        if (issueData) {
          return {
            fillColor: getColor(issueData.primary),
            weight: 2,
            opacity: 1,
            color: 'white',
            fillOpacity: 0.7
          };
        } else {
          return {
            fillColor: '#e0e0e0',
            weight: 1,
            opacity: 0.4,
            color: '#999',
            fillOpacity: 0.3
          };
        }
      }

      // Function to add legend
      function addLegend(map) {
        const legend = L.control({position: 'bottomright'});
        
        legend.onAdd = function (map) {
          const div = L.DomUtil.create('div', 'legend');
          
          div.innerHTML = '<h4>Polling Place Issues</h4>';
          
          const categories = [
            { label: 'Closed Precincts', color: '#d32f2f' },
            { label: 'Moved Precincts', color: '#f57c00' },
            { label: 'Address Issues', color: '#fbc02d' },
            { label: 'No Issues', color: '#e0e0e0' }
          ];
          
          categories.forEach(cat => {
            div.innerHTML += `<div class="legend-item"><i style="background:${cat.color}"></i> ${cat.label}</div>`;
          });
          
          return div;
        };
        
        legend.addTo(map);
      }

      // Initialize Map
      const pollingMap = L.map('polling-map', {
        scrollWheelZoom: false,
        zoomControl: true
      }).setView([32.7364, -89.6678], 7);
      
      L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
        attribution: '&copy; OpenStreetMap contributors'
      }).addTo(pollingMap);

      // Load County GeoJSON
      fetch('https://raw.githubusercontent.com/liampittman/2025-elec-maps/main/assets/data/ms-counties.geojson')
        .then(res => res.json())
        .then(data => {
          const geoJsonLayer = L.geoJSON(data, {
            style: styleCounty,
            onEachFeature: function (feature, layer) {
              const countyName = feature.properties.name || feature.properties.NAME || feature.properties.Name;
              const issueData = countyIssues[countyName];
              const details = countyDetails[countyName];
              
              let popupContent = `<strong>${countyName} County</strong><br>`;
              
              if (issueData && details) {
                popupContent += `<br><strong>Issues:</strong> ${issueData.all.join(', ')}<br>`;
                
                if (details.closed) {
                  popupContent += `<br><strong>Closed Precincts:</strong><br>`;
                  details.closed.forEach(p => {
                    popupContent += `• ${p}<br>`;
                  });
                }
                
                if (details.moved) {
                  popupContent += `<br><strong>Moved Precincts:</strong><br>`;
                  details.moved.forEach(p => {
                    popupContent += `• ${p}<br>`;
                  });
                }
                
                if (details.address) {
                  popupContent += `<br><strong>Address Issues:</strong><br>`;
                  details.address.forEach(p => {
                    popupContent += `• ${p}<br>`;
                  });
                }
              } else {
                popupContent += `<br><em>No polling place issues reported</em>`;
              }
              
              layer.bindPopup(popupContent);
            }
          }).addTo(pollingMap);
          
          pollingMap.fitBounds(geoJsonLayer.getBounds());
          addLegend(pollingMap);
        })
        .catch(err => console.error('Error loading county data:', err));
    });
</script>
