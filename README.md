# scd-Mid-s1

This repository contains two separate projects:

## Q2 — FastBite Campus Food-Court Ordering System

A Dockerized Node.js and Express microservice app with a MongoDB database per service and a static web frontend. Its GitHub Pages demo runs with browser-only sample data; the full stack runs locally with Docker Compose.

- **Run locally:** `cd Q2-cafe-system && docker compose up --build`
- **Frontend:** http://localhost:8080
- **API gateway:** http://localhost:3000
- **Live demo:** [FastBite on GitHub Pages](https://schwifty101.github.io/scd-Mid-s1/cafe/)
- **Details:** [Q2 project README](Q2-cafe-system/README.md)

## Hospital Appointment Management System

A React and Vite dashboard with a separate Express appointment API. GitHub Pages serves the static dashboard in demo mode with sample records. For live API operations, Ubuntu Nginx serves the static frontend by server IP and routes requests to the systemd-managed API.

- **Frontend development:** `cd Hospital-management-system/frontend && npm ci && npm run dev`
- **Backend:** `cd Hospital-management-system/backend && npm install && npm start`
- **Live dashboard:** [Hospital on GitHub Pages](https://schwifty101.github.io/scd-Mid-s1/)
- **Ubuntu setup, routes, and recovery:** [Hospital deployment guide](Hospital-management-system/README.md)
- **Frontend details:** [Hospital frontend README](Hospital-management-system/frontend/README.md)

The Hospital demo API stores appointments in memory; restarting it resets the sample data. See its deployment guide for the Ubuntu service setup.
