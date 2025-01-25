# My Journey Deploying a Java Application on DigitalOcean

This project documents my experience setting up a server on DigitalOcean and deploying a Java application built with Gradle.

![Nexus Diagram](https://github.com/Princeton45/Kubernetes-Resume-Challenge/blob/main/images/diagram.jpg)

## Technologies I Used

*   **DigitalOcean:** For cloud server hosting (a "Droplet").
*   **Linux:** The operating system running on my Droplet.
*   **Java:** The programming language for my application.
*   **Gradle:** My build automation tool for the Java project.

## Project Steps

Here's a summary of the steps I took:

### 1. Setting Up My DigitalOcean Droplet

I created a new Droplet on DigitalOcean, choosing Ubuntu as the operating system and adding my SSH key for secure access.
*   **Image Suggestion:** A screenshot of the Droplet creation page on DigitalOcean.

### 2. Connecting to My Droplet via SSH

Using my Droplet's IP address, I connected to it from my terminal using the `ssh` command.
*   **Image Suggestion:** A screenshot of the terminal showing a successful SSH connection.

### 3. Creating a New Linux User

For security, I created a new non-root user with `sudo` privileges using the `adduser` and `usermod` commands.
*   **Image Suggestion:** A screenshot of the terminal showing the `adduser` command being executed.

### 4. Setting up My Development Environment

I updated the package lists (`sudo apt update`) and installed the Java Development Kit (JDK) and Gradle on the server.
*   **Image Suggestion:** A screenshot of the terminal showing the installation of the JDK.

### 5. Deploying My Java Gradle Application

I transferred my Java project to the Droplet using `scp`, built it with Gradle (`./gradlew build`), and then ran the application using the `java` command.
*   **Image Suggestion:** A screenshot of the terminal showing your application running on the Droplet.

### 6. (Optional) Making My Application Run in the Background

For continuous operation, I could use tools like `systemd`, `tmux`, or `screen` to keep my application running in the background, even after disconnecting.

## Conclusion

This project was a great way to learn about cloud deployment and Linux server administration. I successfully deployed my Java application on DigitalOcean! I hope this documentation is helpful to others starting their own cloud journeys.