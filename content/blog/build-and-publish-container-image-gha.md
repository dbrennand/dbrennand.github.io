---
title: "Build and Publish Container Images using GitHub Actions"
date: 2023-09-09T16:27:02+01:00
draft: false
tags: ["DevOps", "CI/CD", "GitHub Actions", "Containers", "Docker"]
showToc: true
---

And we're back! :wave:

In this blog post we will be looking at building and publishing a container image to GitHub Packages using GitHub Actions!

I recently revisited an old project of mine; now called [speeder](https://github.com/dbrennand/speeder). It's a Python script to monitor your internet speed and send the results to [InfluxDB](https://www.influxdata.com/). The results can then be visualised in [Grafana](https://github.com/dbrennand/speeder#docker-compose-stack---influxdb-and-grafana). I originally created this script during the Coronavirus lockdowns whilst internet usage was high and essential for work. Since that time, I've learned a lot about CI/CD and noticed I hadn't automated the build and publish process of the container image! 😬 Time to fix that! 👷

# What is CI/CD?

Before we get started, let's take a quick look at what CI/CD is. According to ChatGPT:

> Continuous Integration (CI) is the practice of frequently and automatically integrating code changes from multiple developers, leading to early issue detection and smoother teamwork. Continuous Delivery (CD) extends CI by automating the deployment process, allowing for faster and more reliable software releases. The benefits of CI/CD include quicker development cycles, higher software quality, reduced errors, and more frequent and reliable software updates.
>

Thanks ChatGPT! :robot:

In short, CI/CD helps developers to automate the process of building, testing and deploying software. It reduces the time and effort required to complete these processes, and helps ensure software is built and tested consistently and reliably.

# Build and Publish a Container Image using GitHub Actions

Begin by creating the GitHub Actions workflow directory `.github/workflows` in your project:

```bash
mkdir -pv .github/workflows
```

Next, create a `.github/workflows/build.yml` file with the following content:

```yaml
name: Build

on:
  push:
    tags:
      - 'v*'

env:
  REGISTRY: ghcr.io
  # https://docs.github.com/en/actions/learn-github-actions/contexts#github-context
  IMAGE_NAME: ${{ github.repository }}

jobs:
  build-and-push-image:
    runs-on: ubuntu-latest
    # Sets the permissions granted to the `GITHUB_TOKEN` for the actions in this job
    permissions:
      contents: read
      packages: write
    steps:
      - name: Checkout repository
        uses: actions/checkout@v3

      - name: Log in to the Container registry
        uses: docker/login-action@v2
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          # https://docs.github.com/en/actions/security-guides/automatic-token-authentication
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Extract metadata (tags, labels) for Docker
        id: meta
        uses: docker/metadata-action@v4
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}

      - name: Build and push Docker image
        uses: docker/build-push-action@v4
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
```

The above GitHub Actions workflow will perform the following steps 📝:

1. Checkout the repository.
2. Login to the GitHub Packages (`ghcr.io`) container registry.
3. Use git to extract repository metadata to be used for the container image tag(s) and labels.
4. Build the container image and push it to the GitHub Packages container registry.

Now, commit and push the `build.yml` workflow:

```bash
git add .github/workflows/build.yml
git commit -m "chore: build and publish container image"
git push
```

The workflow will only trigger when a `push` event occurs for tags matching the pattern `v*`. To trigger the workflow, create a new tag for your repository following the [semver convention](https://semver.org/):

```bash
# Once PR is merged
git checkout main
git pull
git tag v1.0.0
git push --tags
```

And that's it! 🎉 Once the tag is pushed the workflow will trigger! Here is an example from my [speeder](https://github.com/dbrennand/speeder/actions/runs/6112609864/job/16590449113) project :slight_smile:

# Closing Thoughts

The best part about this workflow is the [docker/metadata-action](https://github.com/docker/metadata-action). It uses git metadata from the repository to create the container image's labels according to the Open Container Initiative [image-spec](https://github.com/opencontainers/image-spec/blob/main/annotations.md#pre-defined-annotation-keys) :heart:

```bash
❯ skopeo inspect docker://ghcr.io/dbrennand/speeder | jq .Labels
{
  "org.opencontainers.image.created": "2023-09-07T16:55:29.525Z",
  "org.opencontainers.image.description": "Python script to monitor your internet speed! 🚀  Periodically run librespeed/speedtest-cli and send results to InfluxDB.",
  "org.opencontainers.image.licenses": "MIT",
  "org.opencontainers.image.revision": "1e9dc09c6a95554ecb74f736029ec09c3ba6910c",
  "org.opencontainers.image.source": "https://github.com/dbrennand/speeder",
  "org.opencontainers.image.title": "speeder",
  "org.opencontainers.image.url": "https://github.com/dbrennand/speeder",
  "org.opencontainers.image.version": "v1.1.0"
}
```

Overall, this GitHub Actions workflow helps save a lot of time and effort. Furthermore, it helps make sure the container image is built and published consistently and reliably every time! :rocket:

That's all folks! Thanks for reading! :wave:

# Referenced Documentation

- https://docs.github.com/en/actions/publishing-packages/publishing-docker-images#publishing-images-to-github-packages
- https://github.com/docker/login-action
- https://github.com/docker/metadata-action
- https://github.com/docker/build-push-action
