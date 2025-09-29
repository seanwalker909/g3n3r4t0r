# PostgreSQL Helm Chart

This Helm chart deploys a PostgreSQL database on a Kubernetes cluster. It includes all necessary resources such as Deployment, Service, PersistentVolumeClaim, and Secrets to ensure a secure and persistent database setup.

## Prerequisites

- Kubernetes 1.12+
- Helm 3.x

## Installation

To install the chart, use the following command:

```bash
helm install my-postgres ./postgres-helm-chart
```

Replace `my-postgres` with your desired release name.

## Configuration

You can customize the deployment by modifying the `values.yaml` file. The following parameters can be configured:

- `image.repository`: The PostgreSQL image repository (default: `postgres`)
- `image.tag`: The PostgreSQL image tag (default: `latest`)
- `postgresUser`: The PostgreSQL user (default: `postgres`)
- `postgresPassword`: The PostgreSQL password (default: `postgres`)
- `postgresDatabase`: The name of the database to create (default: `mydatabase`)
- `persistence.enabled`: Enable persistence using PersistentVolumeClaim (default: `true`)
- `persistence.size`: Size of the PersistentVolumeClaim (default: `1Gi`)

## Accessing the Database

After installation, you can access the PostgreSQL database using the service created by the chart. Use the following command to get the service details:

```bash
kubectl get svc
```

## Uninstalling the Chart

To uninstall the chart, use the following command:

```bash
helm uninstall my-postgres
```

## License

This project is licensed under the MIT License. See the LICENSE file for details.