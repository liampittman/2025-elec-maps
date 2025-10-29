---
title: Mississippi Special Elections 2025
layout: default
---

# Interactive Maps: Mississippi's November 2025 Special Elections

**Author:** William Pittman  
**Date:** October 29, 2025  
**Election Date:** November 4, 2025  
**Last Updated:** October 29, 2025

This project maps Mississippi State House and Senate districts holding special elections on November 4, 2025. Most of these elections stem from court-ordered redistricting following federal litigation over racial gerrymandering. The maps integrate official district boundaries with candidate information and demographic data to provide essential context for understanding these elections.

---

## What Was Created

Two interactive web maps showing:
- All 122 House districts and 52 Senate districts for geographic context
- Highlighted districts holding special elections (6 House, 7 Senate)
- Demographic shading showing percentage of non-white voting age population
- Candidate information and party affiliations accessible via clickable districts

The demographic data is central to understanding why these elections exist—these are districts redrawn because courts found the previous maps diluted Black voting power through packing and cracking.

---

## View the Maps

**[House Districts Map](house_map.html)** - 6 districts with special elections  
**[Senate Districts Map](senate_map.html)** - 7 districts with special elections

---

## Technical Approach

### Data Sources
- **District Boundaries**: Official shapefiles from MARIS (Mississippi Automated Resource Information System), adopted February 2025 following court-ordered redistricting
- **Candidate Information**: Manually compiled from state sources by Mississippi Free Press journalist Heather Harrison
- **Demographic Data**: Voting age population by race/ethnicity, embedded in official shapefiles (2024 estimates)

### Workflow
Raw shapefiles and candidate spreadsheets → Python-based geospatial analysis → GeoJSON export → Interactive web maps deployed via GitHub Pages

### Tech Stack
- **Data Processing**: Python (GeoPandas, Pandas) for joining candidate data to district boundaries and calculating demographic metrics
- **Output Format**: GeoJSON for web-native spatial data
- **Web Mapping**: Leaflet.js for interactive visualization
- **Deployment**: Jekyll static site on GitHub Pages

---

**Election Date**: November 4, 2025  
**Last Updated**: October 29, 2025
