# frappe_docker_setup

This project provides a customizable Docker setup for Frappe/ERPNext applications. It allows you to build, extend, and deploy Frappe-based apps using Docker images with custom configurations and app lists.

## Project Structure

- `Dockerfile.base`: Defines the base Docker image with core dependencies for Frappe/ERPNext.
- `Dockerfile.custom`: Extends the base image to add custom apps or configurations.
- `resources/base/apps.json`: Lists the core apps to be installed in the base image.
- `resources/custom/apps.json`: Lists custom apps to be included in the custom image.
- `resources/core/nginx/`: Contains Nginx configuration and entrypoint scripts for serving the application.

## Usage

1. **Build the base image:**
	```sh
	docker build -f Dockerfile.base -t frappe-base .
	```
2. **Build the custom image:**
	```sh
	docker build -f Dockerfile.custom -t frappe-custom .
	```
3. **Configure apps:**
	- Edit `resources/base/apps.json` and `resources/custom/apps.json` to specify which Frappe/ERPNext apps to install.
4. **Nginx setup:**
	- The Nginx entrypoint and template in `resources/core/nginx/` are used to configure the web server at container startup.

## Notes
- This setup is intended for advanced users who want to customize their Frappe/ERPNext Docker deployments.
- Make sure to update the app lists and Nginx configuration as needed for your environment.

## Argument Passing & Dynamic Configuration

### Dockerfile Arguments
Both `Dockerfile.base` and `Dockerfile.custom` can accept build-time arguments using the `ARG` and `ENV` instructions. You can pass variables (such as Frappe/ERPNext version, custom app names, or other settings) at build time with:

```sh
docker build --build-arg ARG_NAME=value ...
```

### Dynamic App Installation
- The `apps.json` files in `resources/base` and `resources/custom` define which Frappe/ERPNext apps are installed in each image.
- During the build, scripts or Dockerfile commands read these files and install the listed apps, making the process flexible and easy to customize for different environments.

### Dynamic Nginx Configuration
- The Nginx setup uses a template (`nginx-template.conf`) and an entrypoint script (`nginx-entrypoint.sh`).
- At container startup, the entrypoint script can use environment variables or arguments to generate the final Nginx config, adapting to different deployment scenarios (hostnames, ports, etc.).

### General Dynamicity
- By separating base and custom resources, and using templated configs and app lists, you can build images for different needs without changing the core Dockerfiles.
- This modular approach supports dynamic builds and deployments by simply changing arguments, environment variables, or config files.
