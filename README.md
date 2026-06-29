# Compass App

A web application that uses a compass to guide you to the nearest pubs, shops, monuments, and interesting places.

## Features
- **Real-time Geolocation:** Uses the browser's Geolocation API to track your current coordinates.
- **Distance & Azimuth Calculation:** Automatically computes the exact distance (using the Haversine formula) and the bearing/azimuth to your target destination.
- **OpenStreetMap Integration:** Dynamically fetches nearby points of interest within a 2 km radius using the Overpass API.

## Tech Stack
- **Vue 3** (Composition API)
- **Vite**

## Recommended Extensions
- **Vue - Official (Volar)** for VS Code development.

## Installation & Setup

Make sure you have [Node.js](https://nodejs.org/) installed on your machine.

1. Navigate to the project directory:
   ```bash
   cd kompas-app
   Install the required dependencies:

Bash
npm install
Start the local development server:

Bash
npm run dev
Build the application for production:

Bash
npm run build
