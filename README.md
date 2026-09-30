# Presto on MinIO: Air-Quality Analytics on Kubernetes

A small data lake deployed on a local Kubernetes cluster (Minikube). Air-quality readings stored
as CSV in MinIO object storage are registered as tables in a Hive Metastore, queried with SQL
through Presto, and charted in Apache Superset.

![Average AQI](Visualization/avg_aqi.jpg)

## Architecture

| Component | Role | Deployed from |
|---|---|---|
| MinIO | S3-compatible object storage holding the raw CSV data | `Minio/minio-dev.yaml` |
| Hive Metastore | Table schemas and locations for data in MinIO | `Hive_metastore/` |
| Presto 0.280 | Distributed SQL engine, using the Hive connector over S3 | `Presto/` |
| Apache Superset | Dashboards on top of Presto | `superset_dev.yaml` |

Each component runs in its own namespace. `pv.yaml` provisions the persistent volumes, and
`presto.py` creates the `weather` table over the CSV files with the Presto Python client.

The dataset has readings for AQI, CO, NO, NO2, O3, SO2, PM2.5, PM10 and NH3 by place and time.
Charts produced from it are in `Visualization/`.

## Running

With Minikube and kubectl installed:

```bash
kubectl apply -f pv.yaml
kubectl apply -f Minio/minio-dev.yaml
kubectl apply -f Hive_metastore/hive_dev.yaml
kubectl apply -f Presto/presto_dev.yaml
kubectl apply -f superset_dev.yaml
./setup_cluster        # lists the pods and service URLs for each component
```

Then upload the CSV data to the `test` bucket in MinIO and run `python presto.py` to create the
table. The service addresses in `presto.py` and
`Presto/presto-server-0.280/etc/catalog/hive.properties` are for the
default Minikube IP and need updating for other clusters.

## Team

Built with [@Keshavaram](https://github.com/Keshavaram) as part of the HPE CTY program.
