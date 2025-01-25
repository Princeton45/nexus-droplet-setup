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


6. **Access Nexus:** Once Nexus was up, I opened my web browser and navigated to `http:http://159.223.109.89//:8081`. 

![access-nexus](https://github.com/Princeton45/nexus-droplet-setup/blob/main/images/access-nexus.png)


### 3. Java Gradle Project: Build and Upload

Now for the fun part – getting my Java project artifacts into Nexus. I started with a simple Gradle project.

1. **Creating Nexus user:** In this step, I need to create a Nexus user in the Nexus application because when connecting Gradle and Maven to Nexus, we don't want to use the default admin credentials.

The Nexus user will only have permissions to upload artifact files to particular Nexus repositories.

I created a custom `nx-java` role with the required permissions.

![role](https://github.com/Princeton45/nexus-droplet-setup/blob/main/images/role.png)


2. **Configuring Maven Gradle to connect to Nexus:** In this step, I need to configure  Gradle to connect to Nexus via the Nexus Repo URL & Credentials

3. **Create a Simple Project:** I created a basic Java project with a `build.gradle` file.

    *   **Picture Suggestion:** A screenshot of your project directory structure and/or your simple Java code.
        *   **Caption:** "My humble Java project, ready to be built and shared."

2. **Configure `build.gradle`:** I added the `maven-publish` plugin to my `build.gradle` and configured it to publish to my Nexus repository.

```gradle
    plugins {
    id 'java'
    id 'org.springframework.boot' version '3.1.0-SNAPSHOT'
    id 'io.spring.dependency-management' version '1.1.0'
}

group 'com.example'
version '1.0-SNAPSHOT'
sourceCompatibility = 17

apply plugin: 'maven-publish'

publishing {
    publications {
        maven(MavenPublication) {
            artifact("build/libs/my-app-$version"+".jar"){
                extension 'jar'
            }
        }
    }

    repositories {
        maven {
            name 'nexus'
            url "http://159.223.109.89:8081/repository/maven-snapshots/" 
            allowInsecureProtocol = true
            credentials {
                username project.repoUser
                password project.repoPassword
            }
        }
    }
}
```

3. **Build and Publish:** I ran the following Gradle commands:

    ```bash
    gradle build
    gradlew publish
    ```
![gradle-publish](https://github.com/Princeton45/nexus-droplet-setup/blob/main/images/gradle-publish.png)


4. **Verify in Nexus:** I went back to my Nexus UI, browsed the repositories, and there it was, in the `maven-snapshots` repository just as specified.

![gradle-repo](https://github.com/Princeton45/nexus-droplet-setup/blob/main/images/gradle-repo.png)


### 4. Java Maven Project: Build and Upload

The process for Maven is quite similar.

1. **Create a Simple Project:**  I set up a basic Java project with a `pom.xml` file.

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
   <settings> 
    <servers>
        <server>
            <id>nexus-snapshots</id>
            <username><your_nexus_username></username>
            <password><your_nexus_password></password>
        </server>
    </servers>
   </settings> 
    ```

4. **Build and Deploy:** I used the following Maven commands:

    ```bash
    mvn package
    mvn deploy
    ```
![maven-deploy](https://github.com/Princeton45/nexus-droplet-setup/blob/main/images/maven-deploy.png)
    

5. **Verify in Nexus:** Just like with Gradle, I checked Nexus and found my Maven-built artifact happily residing in the repository.

![maven-nexus](https://github.com/Princeton45/nexus-droplet-setup/blob/main/images/maven-nexus.png)
