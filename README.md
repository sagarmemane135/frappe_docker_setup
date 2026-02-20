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
