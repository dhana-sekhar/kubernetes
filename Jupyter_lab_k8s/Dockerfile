# Use the official Ubuntu as the base image
FROM ubuntu:latest

# Set environment variables to prevent interactive installation prompts
# ENV DEBIAN_FRONTEND=noninteractive

# Update and install essential packages
RUN apt-get update && apt-get install -y \
    python3 \
    python3-pip \
    python3-dev \
    build-essential \
    && apt-get clean

# Install JupyterLab using pip
RUN pip install jupyterlab --break-system-packages

# Set the working directory inside the container
WORKDIR /workspace

# Expose the port JupyterLab will run on
EXPOSE 8888

# Command to run JupyterLab
CMD ["jupyter", "lab", "--ip=0.0.0.0", "--allow-root", "--no-browser", "--port=8888"]
