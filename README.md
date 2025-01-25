# Running Nexus on Droplet and Publishing Artifact to Nexus

In this project, I will be setting up a Nexus repository on a DigitalOcean server. Then, I will publish Java artifacts to it using Gradle and Maven build tools.

![Nexus Diagram](https://github.com/Princeton45/nexus-droplet-setup/blob/main/images/diagram.png)

## Technologies

*   **Nexus Repository Manager:**
*   **DigitalOcean:** 
*   **Linux (Ubuntu):** 
*   **Java:**
*   **Gradle:** 
*   **Maven:** 

## Step-by-Step Adventure

### 1. Setting Up My DigitalOcean Droplet

First things first, I needed a server. I went to DigitalOcean and spun up a new Droplet. I chose Ubuntu as the OS because it's familiar and well-supported.

![Droplet](https://github.com/Princeton45/nexus-droplet-setup/blob/main/images/droplet.png)

Once the Droplet was live, I SSH'd into it using my terminal.

![Droplet](https://github.com/Princeton45/nexus-droplet-setup/blob/main/images/ssh.png)

### 2. Installing and Configuring Nexus

With my server ready, it was time to bring in Nexus. I followed the official Nexus installation guide. Here's the gist:

1. **Download Nexus:** I grabbed the latest version from the Sonatype website.

    https://help.sonatype.com/en/download.html

2. **Extract & Run:** Unpacked the downloaded file and ran the Nexus startup script. I am going to extract the contents in the /opt folder.

    ```bash
    cd /opt
    wget https://download.sonatype.com/nexus/3/nexus-3.76.1-01-unix.tar.gz 
    tar -zxvf nexus-3.76.1-01-unix.tar.gz
    ```

3. **Nexus Service Account:** I then created a nexus service account called `nexus-princeton` on the Linux server and made the service account an owner of the nexus and sonatype directory

![serviceacc](https://github.com/Princeton45/nexus-droplet-setup/blob/main/images/serviceacc.png)

4. **Setting Nexus Configuration:** Next I set the `nexus-3.76.1-01/bin/nexus.rc`  configuration file so that it will run as a nexus user.

![runas](https://github.com/Princeton45/nexus-droplet-setup/blob/main/images/runasuser.png)

5. **Starting Nexus service:** I ran the command below to start the nexus service.

`/opt/nexus-3.76.1-01/bin/nexus start`

![nexusrunning](https://github.com/Princeton45/nexus-droplet-setup/blob/main/images/nexus-running.png)


5. **Access Nexus:** Once Nexus was up, I opened my web browser and navigated to `http:http://159.223.109.89//:8081`. 

![access-nexus](https://github.com/Princeton45/nexus-droplet-setup/blob/main/images/access-nexus.png)

4. **Initial Login and Password Change:** I retrieved the initial admin password from the server as indicated on the first login page. Used this to login, and immediately changed the password to something more secure.

### 3. Creating a New Nexus User

I didn't want to use the admin user for everything, so I created a new user specifically for uploading artifacts.

1. **Navigate to User Management:** I went to the "Server Administration and Configuration" section (the gear icon) and then to "User" under the "Security" section.
2. **Create User:** Clicked on "Create user".
3. **Set Permissions:** I gave the user the appropriate roles (e.g., `nx-repository-view-*-*-*`, `nx-repository-admin-*-*-*`). You can be more specific, but this allows the user to do everything related to repositories.

    *   **Picture Suggestion:** A screenshot of the Nexus user creation page, showing the fields filled in and the roles assigned.
        *   **Caption:** "Crafting a new user with just the right powers – no admin overkill here!"

### 4. Java Gradle Project: Build and Upload

Now for the fun part – getting my Java project artifacts into Nexus. I started with a simple Gradle project.

1. **Create a Simple Project:** I created a basic Java project with a `build.gradle` file.

    *   **Picture Suggestion:** A screenshot of your project directory structure and/or your simple Java code.
        *   **Caption:** "My humble Java project, ready to be built and shared."

2. **Configure `build.gradle`:** I added the `maven-publish` plugin to my `build.gradle` and configured it to publish to my Nexus repository. Remember to replace placeholders with your Nexus URL and credentials.

    ```gradle
    plugins {
        id 'java'
        id 'maven-publish'
    }

    group = 'com.example'
    version = '1.0'

    publishing {
        publications {
            myPublication(MavenPublication) {
                from components.java
            }
        }
        repositories {
            maven {
                url 'http://<your_droplet_ip>:8081/repository/maven-releases/' // Your Nexus repo URL
                credentials {
                    username = '<your_nexus_username>'
                    password = '<your_nexus_password>'
                }
            }
        }
    }
    ```

3. **Build and Publish:** I ran the following Gradle commands:

    ```bash
    ./gradlew build
    ./gradlew publish
    ```

    *   **Picture Suggestion:** A screenshot of your terminal showing the successful output of the `./gradlew publish` command.
        *   **Caption:** "Gradle doing its magic – building and publishing like a champ!"

4. **Verify in Nexus:** I went back to my Nexus UI, browsed the repositories, and there it was – my artifact, safe and sound.

    *   **Picture Suggestion:** A screenshot of your Nexus UI showing your published artifact in the repository.
        *   **Caption:** "And there's my artifact, chilling in Nexus. Mission accomplished!"

### 5. Java Maven Project: Build and Upload

The process for Maven is quite similar.

1. **Create a Simple Project:**  I set up a basic Java project with a `pom.xml` file.

    *   **Picture Suggestion:** A screenshot of your Maven project directory structure and/or your simple Java code.
        *   **Caption:** "Maven project reporting for duty! Simple and ready to go."

2. **Configure `pom.xml`:** I added the `distributionManagement` section to my `pom.xml` to point to my Nexus repository. Again, replace the placeholders.

    ```xml
    <project>
        ...
        <distributionManagement>
            <repository>
                <id>nexus-releases</id>
                <url>http://<your_droplet_ip>:8081/repository/maven-releases/</url>
            </repository>
        </distributionManagement>
        ...
    </project>
    ```

3. **Configure `settings.xml`:**  I added my Nexus credentials to my Maven `settings.xml` file (usually found in `~/.m2/`). This is crucial to avoid putting credentials in every project.

    ```xml
    <servers>
        <server>
            <id>nexus-releases</id>
            <username><your_nexus_username></username>
            <password><your_nexus_password></password>
        </server>
    </servers>
    ```

4. **Build and Deploy:** I used the following Maven commands:

    ```bash
    mvn clean package
    mvn deploy
    ```

    *   **Picture Suggestion:** A screenshot of your terminal showing the successful output of the `mvn deploy` command.
        *   **Caption:** "Maven in action – deploying my artifact to Nexus with style."

5. **Verify in Nexus:** Just like with Gradle, I checked Nexus and found my Maven-built artifact happily residing in the repository.

    *   **Picture Suggestion:** A screenshot of your Nexus UI showing your published Maven artifact.
        *   **Caption:** "Another one in the books! My Maven artifact is now part of the Nexus family."

## Conclusion

And that's it! I successfully set up Nexus on a Droplet, created a user, and published artifacts from both Gradle and Maven projects. It was a great learning experience, and I hope this guide helps you on your own journey into the world of artifact management. Feel free to reach out if you have any questions. Happy coding!