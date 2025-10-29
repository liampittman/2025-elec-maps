---
layout: default
title: Senate Map
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
  .legend .no-election {
    color: #999 !important;
    font-style: italic !important;
    margin-top: 5px !important;
  }
  .legend .litigation-marker {
    margin-top: 10px !important;
    padding-top: 10px !important;
    border-top: 1px solid #ddd !important;
  }
  .legend .litigation-box {
    width: 18px !important;
    height: 18px !important;
    float: left !important;
    margin-right: 8px !important;
    border: 3px solid #e63946 !important;
    background: rgba(230, 57, 70, 0.2) !important;
    box-shadow: 0 0 8px rgba(230, 57, 70, 0.6) !important;
  }
  @keyframes pulse-glow {
    0%, 100% {
      box-shadow: 0 0 8px rgba(255, 107, 53, 0.6);
    }
    50% {
      box-shadow: 0 0 15px rgba(255, 107, 53, 0.9);
    }
  }
  .leaflet-interactive.litigation-district {
    animation: pulse-glow 2s ease-in-out infinite;
  }
</style>

<div class="map-container">
  <div id="senate-map" class="map"></div>
</div>

<script src="https://unpkg.com/leaflet/dist/leaflet.js"></script>
<script>
    document.addEventListener("DOMContentLoaded", function () {
      
      // Function to get color based on F_18_NonWHT value
      function getColor(value) {
          // Clamp value between 28 and 68
          const clampedValue = Math.max(28, Math.min(68, value));
  
          // Normalize to 0-1 range
          const normalized = (clampedValue - 28) / (68 - 28);
  
          // Interpolate between light (#fffcd4) and dark purple (#6c0175)
          const r = Math.round(255 - (255 - 108) * normalized);
          const g = Math.round(252 - (252 - 1) * normalized);
          const b = Math.round(212 - (212 - 117) * normalized);
  
          return `rgb(${r}, ${g}, ${b})`;
        }

      // Function to parse the F_18_NonWHT value - handle different formats
      function parseNonWhiteValue(value) {
        if (value === null || value === undefined || value === '') {
          return 0;
        }
        
        // If it's already a number
        if (typeof value === 'number') {
          // Check if it's a decimal (0-1) or percentage (0-100)
          return value > 1 ? value : value * 100;
        }
        
        // If it's a string
        if (typeof value === 'string') {
          // Remove any % signs
          value = value.replace('%', '').trim();
          const num = parseFloat(value);
          
          if (isNaN(num)) {
            return 0;
          }
          
          // Check if it's a decimal (0-1) or percentage (0-100)
          return num > 1 ? num : num * 100;
        }
        
        return 0;
      }

      // Function to style districts
      function styleDistrict(feature) {
        const props = feature.properties;
        const c1_name = props.c1_name || props.C1_NAME || props.C1_Name || props['c1_name'];
        const f18NonWhtRaw = props.F_18_NonWHT || props.f_18_nonwht || props.F_18_NONWHT || props['F_18_NonWHT'];
        
        const hasSpecialElection = c1_name !== null && 
                                   c1_name !== undefined &&
                                   c1_name !== '';
        
        if (hasSpecialElection) {
          const nonWhitePercent = parseNonWhiteValue(f18NonWhtRaw);
          const color = getColor(nonWhitePercent);
          const reason = props.reason || props.Reason || props.REASON;
          const isLitigation = reason === 'Litigation';
          
          return {
            fillColor: color,
            weight: isLitigation ? 2 : 2,
            opacity: 1,
            color: isLitigation ? '#e63946' : 'white',
            fillOpacity: 0.7,
            className: isLitigation ? 'litigation-district' : ''
          };
        } else {
          return {
            fillColor: 'none',
            weight: 1,
            opacity: 0.4,
            color: '#999',
            fillOpacity: 0
          };
        }
      }

      // Function to add legend
      function addLegend(map) {
  const legend = L.control({position: 'bottomright'});
  
          legend.onAdd = function (map) {
            const div = L.DomUtil.create('div', 'legend');
    
            div.innerHTML = '<h4>% Non-White Voting Age Population<br>(Special Election Districts)</h4>';
    
            // Create gradient bar
            div.innerHTML += '<div style="background: linear-gradient(to right, #fffcd4, #e8d5c4, #d1aeb4, #ba87a4, #a36094, #8c3984, #751274, #6c0175); height: 20px; width: 200px; border: 1px solid #999; margin-bottom: 5px;"></div>';
            div.innerHTML += '<div style="display: flex; justify-content: space-between; width: 200px !important; font-size: 11px;">';
            div.innerHTML += '<span>28% &ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp; 68%</span>';
            div.innerHTML += '</div>';
            
            div.innerHTML += '<div class="litigation-marker"><div class="litigation-box"></div> Litigation-Related Election</div>';
    
            return div;
          };
  
          legend.addTo(map);
        }
      // Initialize Senate Map
      const senateMap = L.map('senate-map', {
        scrollWheelZoom: false,
        zoomControl: true
      }).setView([32.3547, -89.3985], 7);
      
      L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
        attribution: '&copy; OpenStreetMap contributors'
      }).addTo(senateMap);

      // Load Senate Districts
      fetch('assets/data/senate_districts.geojson')
        .then(res => res.json())
        .then(data => {
          const geoJsonLayer = L.geoJSON(data, {
            style: styleDistrict,
            onEachFeature: function (feature, layer) {
              const props = feature.properties;
              const c1_name = props.c1_name || props.C1_NAME || props.C1_Name;
              const f18NonWhtRaw = props.F_18_NonWHT || props.f_18_nonwht || props.F_18_NONWHT;
  
              const hasSpecialElection = c1_name !== null && 
                            c1_name !== undefined &&
                            c1_name !== '';
  
              let popupContent = `<strong>Senate District ${props.name || 'N/A'}</strong><br>`;
  
              if (hasSpecialElection) {
                  const nonWhitePercent = parseNonWhiteValue(f18NonWhtRaw);
                  const reason = props.reason || props.Reason || props.REASON;

                  popupContent += `Non-White Voting Age Population: ${nonWhitePercent.toFixed(1)}%<br>`;
                  
                  if (reason === 'Litigation') {
                    popupContent += `<em>Election Type: Litigation-Related</em><br>`;
                  } else if (reason === 'Regular Special Election') {
                    popupContent += `<em>Election Type: Regular Special Election</em><br>`;
                  }
                  
                  popupContent += `<br><strong>Candidate(s):</strong><br>`;
  
                  // Add all candidates
                  for (let i = 1; i <= 7; i++) {
                    const candidateName = props[`c${i}_name`];
                    const candidateParty = props[`c${i}_party`];
    
                    if (candidateName && candidateName !== null && candidateName !== '') {
                      popupContent += `• ${candidateName}`;
                      
                      if (candidateParty && candidateParty !== null && candidateParty !== '') {
                        popupContent += `, ${candidateParty}`;
                      }
      
                      // Only check incumbency for candidate 1
                      if (i === 1 && props.c1_incumbancy === 'True') {
                        popupContent += ` (Incumbent)`;
                      }
      
                      popupContent += `<br>`;
                    }
                  }
                  
                  // Add link to story if available
                  if (props.link && props.link !== null && props.link !== '') {
                    popupContent += `<br>For more information about this district's special election, read the story <a href="${props.link}" target="_blank">here</a>.`;
                  }
                } else {
                  popupContent += `No special election`;
                }
  
              layer.bindPopup(popupContent);
            }
          }).addTo(senateMap);
          
          senateMap.fitBounds(geoJsonLayer.getBounds());
          addLegend(senateMap);
        })
        .catch(err => console.error('Error loading senate districts:', err));
    });
</script>
