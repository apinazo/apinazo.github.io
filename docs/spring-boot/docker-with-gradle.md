# Creating Docker images and containers with Gradle

## Palantir

Palantir is very well-known and most used plugin for managing Docker images and containers with Gradle.

In this example, Palantir will be the tool of choice.

### Enabling Palantir

First of all, add the dependency:

    buildscript {
        ...
        dependencies {
            ...
            // https://github.com/palantir/gradle-docker
            classpath "gradle.plugin.com.palantir.gradle.docker:gradle-docker:0.21.0"
        }
    }
    
Then apply the plugin:    
    
    apply plugin: "com.palantir.docker" // Create Docker images and containers.


### Creating the image
    
The task 'docker' is the one responsible of creating the image. This is a very simple configuration example:    
    
    docker {
        name imageName
        files bootJar.outputs
        copySpec.into("build/libs")
    }

The interesting part is the task:

* By default, Palantir looks for a Dockerfile in the root directory of the project.
* `files` is adding the output files from the bootJar task, to the Palantir context.
* Then, `copySpec` is copying that files into the `build/libs' directory of that context. 

The example assumes the project uses the Spring Boot plugin which creates a JAR of the app in the 'build/libs`directory.

Now, to create a Docker image, just execute:

    gradlew docker

The image will be created in the local Docker instance.

For further information regarding advanced configurations, please go to the official documentation.

## References

* [Palantir](https://github.com/palantir/gradle-docker).