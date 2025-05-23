---
title: Docker Provider
description: Deploy and manage Docker resources using Alchemy
---

# Docker Provider

The Docker provider allows you to create, manage, and orchestrate Docker resources directly from your Alchemy applications. With this provider, you can pull images, run containers, create networks, and more, all using the familiar Alchemy Resource syntax.

## Resources

The Docker provider includes the following resources:

- [DockerRemoteImage](./remote-image.md) - Pull and manage Docker images
- [DockerImage](./image.md) - Build Docker images from local Dockerfiles
- [DockerContainer](./container.md) - Run and manage Docker containers
- [DockerNetwork](./network.md) - Create and manage Docker networks
- [DockerVolume](./volume.md) - Create and manage persistent Docker volumes

## Example

Here's a complete example of using the Docker provider to create a web application with Redis, custom images, and persistent volumes:

```typescript
import { DockerContainer, DockerRemoteImage, DockerNetwork, DockerImage, DockerVolume } from "alchemy/docker";

// Create a Docker network
const network = await DockerNetwork("app-network", {
  name: "my-application-network"
});

// Create a persistent volume for Redis data
const redisVolume = await DockerVolume("redis-data", {
  name: "redis-data",
  labels: [
    { name: "app", value: "my-application" },
    { name: "service", value: "redis" }
  ]
});

// Pull Redis image
const redisImage = await DockerRemoteImage("redis-image", {
  name: "redis",
  tag: "alpine"
});

// Run Redis container with persistent volume
const redis = await DockerContainer("redis", {
  image: redisImage.imageRef,
  name: "redis",
  networks: [{ name: network.name }],
  volumes: [
    {
      hostPath: redisVolume.name,
      containerPath: "/data"
    }
  ],
  start: true
});

// Build a custom application image from local Dockerfile
const appImage = await DockerImage("app-image", {
  name: "my-web-app",
  tag: "latest",
  build: {
    context: "./app",
    buildArgs: {
      NODE_ENV: "production"
    }
  }
});

// Create a volume for application logs
const logsVolume = await DockerVolume("logs-volume", {
  name: "app-logs",
  labels: {
    "com.example.environment": "production",
    "com.example.backup": "daily"
  }
});

// Run the application container
const app = await DockerContainer("app", {
  image: appImage,  // Using the custom built image
  name: "web-app",
  ports: [{ external: 3000, internal: 3000 }],
  networks: [{ name: network.name }],
  volumes: [
    {
      hostPath: logsVolume.name,
      containerPath: "/app/logs"
    }
  ],
  environment: {
    REDIS_HOST: "redis",
    NODE_ENV: "production"
  },
  restart: "always",
  start: true
});

// Output the URL
export const url = `http://localhost:3000`;
```

## Additional Resources

For more complex examples, see the [Docker Example](https://github.com/sam-goodwin/alchemy/tree/main/examples/docker) in the Alchemy repository.
