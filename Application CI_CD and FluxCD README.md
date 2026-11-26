# Application CI/CD and GitOps Deployment

## Overview

GitHub Actions handles CI: build, test, security scans, Docker image creation, and push to Azure Container Registry (ACR). FluxCD handles CD by reconciling Helm releases from the GitOps repository into AKS.

> We use GitHub Actions for CI and FluxCD for GitOps-based CD. A pull request runs tests, SonarQube, Black Duck, Gitleaks, and container scanning. After the gates pass, the pipeline builds a multi-stage Docker image and pushes an immutable version to ACR. The approved image tag is updated in the GitOps repository, and FluxCD deploys the Helm release to AKS. We promote the same image through all environments and roll back by reverting the GitOps commit.

## Delivery Flow

```text
Developer PR -> Unit Tests -> SonarQube -> Black Duck -> Gitleaks
             -> Docker Build/Scan -> Push Image to ACR
             -> Update GitOps Image Tag -> FluxCD -> Helm -> AKS
```

## CI Security Gates

- **SonarQube:** source-code quality, bugs, vulnerabilities, coverage, and quality gate.
- **Black Duck:** open-source dependency CVEs, licences, and policy violations.
- **Gitleaks:** detects committed passwords, tokens, keys, and secrets.
- **Container scan:** blocks images with unapproved critical/high vulnerabilities.
- Use immutable tags such as `2.4.1-<git-sha>`; do not deploy only `latest`.

## Sample Dockerfile 1: Angular + NGINX

```dockerfile
# Build stage
FROM node:20-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build -- --configuration production

# Runtime stage
FROM nginx:1.27-alpine
RUN rm -rf /usr/share/nginx/html/*
COPY --from=build /app/dist/<angular-app>/browser/ /usr/share/nginx/html/
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 8080
USER nginx
CMD ["nginx", "-g", "daemon off;"]
```

> Replace `<angular-app>` with the actual Angular output directory. Ensure `nginx.conf` listens on port `8080` when running as the non-root `nginx` user.

## Sample Dockerfile 2: Java JAR + JRE

```dockerfile
# Build stage
FROM maven:3.9-eclipse-temurin-21-alpine AS build
WORKDIR /app
COPY pom.xml ./
RUN mvn -B dependency:go-offline
COPY src ./src
RUN mvn -B clean package -DskipTests

# Runtime stage
FROM eclipse-temurin:21-jre-alpine
RUN addgroup -S app && adduser -S app -G app
WORKDIR /app
COPY --from=build /app/target/*.jar app.jar
USER app
EXPOSE 8080
ENTRYPOINT ["java", "-XX:MaxRAMPercentage=75.0", "-jar", "/app/app.jar"]
```

> Unit tests should normally run in GitHub Actions before the image build. Remove `-DskipTests` if tests must also run inside the Docker build.

## FluxCD Deployment

The CI workflow does not directly run `kubectl` against AKS. After publishing the image, it raises a PR to update the immutable image tag in the GitOps repository.

```text
ACR image -> GitOps PR -> Approval -> Merge
          -> Flux source-controller detects change
          -> helm-controller reconciles HelmRelease
          -> AKS rolling deployment -> Health validation
```

Example GitOps layout:

```text
gitops/
├── dev/docgen-api/
├── qa/docgen-api/
├── stage/docgen-api/
└── prod/docgen-api/
```

Promotion uses the **same image digest/tag** across Dev, QA, Stage, and Production. Production requires PR approval, GitHub Environment approval, change approval, and a rollback plan.

Validation:

```bash
flux get sources git -A
flux get helmreleases -A
kubectl rollout status deployment/docgen-api -n <namespace>
kubectl get pods,events -n <namespace>
```

Rollback is performed by reverting the GitOps commit or restoring the previously validated image tag. Flux then reconciles AKS back to that version.

## Reusable Application Workflow

```text
central-workflows/.github/workflows/
├── application-ci.yml
├── sonarqube.yml
├── blackduck.yml
├── docker-build.yml
└── gitops-update.yml
```

Each application repository calls a versioned reusable workflow and passes:

- Application name and build type
- Java/Node version
- SonarQube and Black Duck project keys
- Dockerfile and ACR repository
- GitOps repository path and target environment

Pin reusable workflows to a release tag or commit SHA, grant minimum `GITHUB_TOKEN` permissions, and use OIDC instead of long-lived Azure credentials.

## Interview Summary

> We use GitHub Actions for CI and FluxCD for GitOps-based CD. A pull request runs tests, SonarQube, Black Duck, Gitleaks, and container scanning. After the gates pass, the pipeline builds a multi-stage Docker image and pushes an immutable version to ACR. The approved image tag is updated in the GitOps repository, and FluxCD deploys the Helm release to AKS. We promote the same image through all environments and roll back by reverting the GitOps commit.
