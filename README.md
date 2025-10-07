<p align="center">
    <a href="https://ecourse.pockethost.io/" target="_blank" rel="noopener">
        <img src="https://i.ibb.co/Sx7YmY6/ecourse.jpg" alt="eCourse - My Courses" />
    </a>
</p>

eCourse is a self-hosted SPA designed to simplify course creation and management, some of the features include:

- ability to create video and text based content
- ability to assign courses to users
- ability to track users progress

[Demo](https://ecourse.pockethost.io/)

Use the following credentials for testing the demo:

**Username:** `ilyas`  
**Password:** `ecourse123`

## Tech Stack

**UI Framework** - [Svelte](https://svelte.dev/)

**CSS** - [TailwindCSS](https://tailwindcss.com/)

**Icons** - [Iconify](https://iconify.design/)

**Backend** - [PocketBase](https://pocketbase.io/)

**Hosting** - [PocketHost](https://pockethost.io/)

## Getting Started

Get started by running the project locally, simply follow these steps:

1. Clone/download the repo

2. Grab the PocketBase executable for your OS from: https://pocketbase.io/docs/ and drop it at the root of the `pb` folder.

3. Start the PocketBase server

```bash
cd pb
./pocketbase serve
```

4. Start the Vite server

```bash
cd ui
npm install && npm run dev
```

## Customization

App name, logo, and colors can be customized using the `customize.json` file.

## Deployment

One neat thing about PocketBase is that it can also serve our static frontend assets. to do that simply follow these steps:

1. Add the server URL where your PocketBase instance is hosted to `VITE_PROD_PB_URL` in the `.env` file

2. Build a production-ready bundle

```bash
cd ui
npm run build
```

3. Copy the contents of the `dist` folder over to `pb_public`

### Using Docker

You can use the following Dockerfile to automate the steps above:

```dockerfile
# Use a Node.js base image (updated to avoid EOL repos)
FROM node:20-bookworm

# Set the working directory
WORKDIR /app

# Copy application files
COPY . /app

# Ensure entrypoint script is executable
RUN chmod +x /app/entrypoint.sh

# Install unzip utility (now works with Bookworm repos)
RUN apt-get update && apt-get install -y unzip && rm -rf /var/lib/apt/lists/*

# Download and extract PocketBase (overwrite existing files without prompting)
ARG PB_VERSION=0.22.20
RUN mkdir -p /app/pb \
    && echo "Downloading PocketBase version ${PB_VERSION}" \
    && wget https://github.com/pocketbase/pocketbase/releases/download/v${PB_VERSION}/pocketbase_${PB_VERSION}_linux_amd64.zip -O /tmp/pb.zip \
    && unzip -o /tmp/pb.zip -d /app/pb \
    && rm /tmp/pb.zip \
    && ls -la /app/pb

# Switch to the UI directory
WORKDIR /app/ui

# Create .env if missing and update VITE_PROD_PB_URL
# replace with details from your server
RUN echo "VITE_PROD_PB_URL=http://mysite.lan:5211" > .env \
    && sed -i 's/^VITE_PROD_PB_URL=.*/VITE_PROD_PB_URL=http:\/\/mysite.lan:5211/' .env \
    && npm install \
    && npm run build \
    && mkdir -p /app/pb/pb_public \
    && mv dist/* /app/pb/pb_public/

# Expose the PocketBase port
EXPOSE 8090

# Use the script to start the application
ENTRYPOINT ["/app/entrypoint.sh"]
```

Then deploy container using built image:

```
version: '3.8'

services:
  ecourse:
    image: local-ecourse:manual
    container_name: ecourse
    volumes:
      - /volume1/data/courses:/app/media:ro
      - /volume1/docker/eCourse:/app/pb/pb_data
    environment:
      - VITE_PROD_PB_URL=http://ecourse.mysite.lan  # I run it through reverse proxy
    ports:
      - "5211:8090"
    restart: unless-stopped
```


## Feedback & Suggestions

Feel free to open an issue/PR if you find any bugs or want to request new features.

## License

Licensed under the MIT License, Copyright © 2024
