# Hospital dashboard frontend

This Vite app builds to static HTML, JavaScript, CSS, and assets. The built files can be hosted on GitHub Pages or served by Ubuntu Nginx; Node.js is only needed to build the files, not to serve them.

## GitHub Pages

The `hospital-pages.yml` workflow builds the dashboard on pushes to `main` and manual runs. In the repository's **Settings → Pages**, select **GitHub Actions** as the build and deployment source. The workflow derives the project-site path from the repository name and publishes the `dist` directory.

The dashboard shows sample doctors and an appointment when its API is unavailable. Creating, changing, and removing appointments requires the separate backend service.

## Local development and static build

```sh
npm ci
npm run dev
```

For an Ubuntu root-site build, run `npm run build` with `VITE_BASE_PATH` unset, then copy `dist/` to the Nginx document root. See [the Hospital deployment guide](../README.md) for Nginx and backend setup.
