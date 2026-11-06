# Hospital appointment system

The frontend builds to static files. Ubuntu Nginx serves it at the server's IP address independently of the API process. The Express API runs as a separate systemd service; Nginx forwards `/api/` and `/health` requests to it on loopback.

## GitHub Pages deployment

1. In the repository's **Settings → Pages**, select **GitHub Actions** as the deployment source.
2. Push to `main` or run **Deploy Hospital dashboard to GitHub Pages** from the Actions tab.
3. Open the Pages URL shown in the deployment environment.

The workflow builds only the frontend and publishes its `dist/` directory. The dashboard remains viewable with sample records when the API is offline; appointment changes require the Ubuntu backend.

## Ubuntu deployment

These commands assume the backend files are copied to `/opt/hospital/backend` and the built static files are copied to `/var/www/hospital`. Build the frontend on a build machine with Node.js and transfer only `dist/` to Ubuntu:

```sh
cd Hospital-management-system/frontend
npm ci
npm run build
rsync -av --delete dist/ ubuntu-user@SERVER_IP:/tmp/hospital-dist/
```

On Ubuntu, install Nginx and a supported Node.js release, then install the backend and static files:

```sh
sudo apt update
sudo apt install -y nginx
sudo mkdir -p /opt/hospital/backend /var/www/hospital
sudo cp -R Hospital-management-system/backend/. /opt/hospital/backend/
sudo npm install --omit=dev --prefix /opt/hospital/backend
sudo rsync -av --delete /tmp/hospital-dist/ /var/www/hospital/
sudo chown -R root:www-data /opt/hospital/backend
sudo chmod -R o-rwx /opt/hospital/backend
sudo chmod -R g+rX /opt/hospital/backend
sudo chmod -R a+rX /var/www/hospital
```

Copy and enable the supplied service and Nginx configuration:

```sh
sudo cp Hospital-management-system/deploy/hospital-api.service /etc/systemd/system/hospital-api.service
sudo cp Hospital-management-system/deploy/hospital-nginx.conf /etc/nginx/sites-available/hospital
sudo ln -sfn /etc/nginx/sites-available/hospital /etc/nginx/sites-enabled/hospital
sudo rm -f /etc/nginx/sites-enabled/default
sudo systemctl daemon-reload
sudo systemctl enable --now hospital-api
sudo nginx -t
sudo systemctl enable --now nginx
sudo systemctl reload nginx
```

If UFW is enabled, allow HTTP traffic with `sudo ufw allow 'Nginx HTTP'`. Open `http://SERVER_IP/` in a browser. Static dashboard files continue to load if Node.js or the API is unavailable.

## API routes and request handling

The browser calls the same host, and Nginx routes API requests to Express on `127.0.0.1:4001`.

| Method | Route | Purpose |
| --- | --- | --- |
| `GET` | `/health` | API health and service status |
| `GET` | `/api/doctors` | List doctors |
| `GET` | `/api/appointments?patient=<name>` | List appointments, optionally filtered by patient name |
| `POST` | `/api/appointments` | Create an appointment with JSON fields `patientName`, `doctorId`, `date`, `time`, and optional `reason` |
| `PATCH` | `/api/appointments/:id` | Change status to `Pending`, `Confirmed`, or `Cancelled` |
| `DELETE` | `/api/appointments/:id` | Remove an appointment |

## Availability and recovery

- Check API status: `sudo systemctl status hospital-api` and `curl http://127.0.0.1:4001/health`.
- Check the public route: `curl http://SERVER_IP/health`.
- Read API logs: `sudo journalctl -u hospital-api -n 100 --no-pager` or follow them with `sudo journalctl -u hospital-api -f`.
- Restart the API after a configuration or dependency issue: `sudo systemctl restart hospital-api`.
- Check/reload Nginx: `sudo nginx -t && sudo systemctl reload nginx`; inspect failures with `sudo journalctl -u nginx -n 100 --no-pager`.
- `hospital-api` starts at boot and systemd retries it after a process failure. Nginx serves the static dashboard separately, so an API outage does not prevent browser access.

The demo API stores appointments in memory. Restarting or recovering the API resets its appointment list to the sample record; persistent recovery requires a database, which is outside this setup.
