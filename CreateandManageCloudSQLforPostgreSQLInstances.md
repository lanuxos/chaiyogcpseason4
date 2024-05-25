# Create and Manage Cloud SQL for PostgreSQL Instances
## Migrate to Cloud SQL for PostgreSQL using Database Migration Service [GSP918]
### Prepare the source database for migration
Compute Engine > VM instances > SSH > Authorize
sudo apt install postgresql-13-pglogical

- Download and apply some additions to the PostgreSQL configuration files (to enable pglogical extension) and restart the postgresql service
sudo su - postgres -c "gsutil cp gs://cloud-training/gsp918/pg_hba_append.conf ."
sudo su - postgres -c "gsutil cp gs://cloud-training/gsp918/postgresql_append.conf ."
sudo su - postgres -c "cat pg_hba_append.conf >> /etc/postgresql/13/main/pg_hba.conf"
sudo su - postgres -c "cat postgresql_append.conf >> /etc/postgresql/13/main/postgresql.conf"
sudo systemctl restart postgresql@13-main

- launch the psql tool
sudo su - postgres
psql

- Add the pglogical database extension to the postgres, orders and gmemegen_db databases
\c postgres;
CREATE EXTENSION pglogical;
\c orders;
CREATE EXTENSION pglogical;
\c gmemegen_db;
CREATE EXTENSION pglogical;

- List the PostgreSQL databases on the server
\l

- Create the database migration user
CREATE USER migration_admin PASSWORD 'DMS_1s_cool!';
ALTER DATABASE orders OWNER TO migration_admin;
ALTER ROLE migration_admin WITH REPLICATION;

- Assign permissions to the migration user
\c postgres;

GRANT USAGE ON SCHEMA pglogical TO migration_admin;
GRANT ALL ON SCHEMA pglogical TO migration_admin;

GRANT SELECT ON pglogical.tables TO migration_admin;
GRANT SELECT ON pglogical.depend TO migration_admin;
GRANT SELECT ON pglogical.local_node TO migration_admin;
GRANT SELECT ON pglogical.local_sync_status TO migration_admin;
GRANT SELECT ON pglogical.node TO migration_admin;
GRANT SELECT ON pglogical.node_interface TO migration_admin;
GRANT SELECT ON pglogical.queue TO migration_admin;
GRANT SELECT ON pglogical.replication_set TO migration_admin;
GRANT SELECT ON pglogical.replication_set_seq TO migration_admin;
GRANT SELECT ON pglogical.replication_set_table TO migration_admin;
GRANT SELECT ON pglogical.sequence_state TO migration_admin;
GRANT SELECT ON pglogical.subscription TO migration_admin;

- grant permissions to the pglogical schema and tables for the orders database
\c orders;

GRANT USAGE ON SCHEMA pglogical TO migration_admin;
GRANT ALL ON SCHEMA pglogical TO migration_admin;

GRANT SELECT ON pglogical.tables TO migration_admin;
GRANT SELECT ON pglogical.depend TO migration_admin;
GRANT SELECT ON pglogical.local_node TO migration_admin;
GRANT SELECT ON pglogical.local_sync_status TO migration_admin;
GRANT SELECT ON pglogical.node TO migration_admin;
GRANT SELECT ON pglogical.node_interface TO migration_admin;
GRANT SELECT ON pglogical.queue TO migration_admin;
GRANT SELECT ON pglogical.replication_set TO migration_admin;
GRANT SELECT ON pglogical.replication_set_seq TO migration_admin;
GRANT SELECT ON pglogical.replication_set_table TO migration_admin;
GRANT SELECT ON pglogical.sequence_state TO migration_admin;
GRANT SELECT ON pglogical.subscription TO migration_admin;

- grant permissions to the public schema and tables for the orders database
GRANT USAGE ON SCHEMA public TO migration_admin;
GRANT ALL ON SCHEMA public TO migration_admin;

GRANT SELECT ON public.distribution_centers TO migration_admin;
GRANT SELECT ON public.inventory_items TO migration_admin;
GRANT SELECT ON public.order_items TO migration_admin;
GRANT SELECT ON public.products TO migration_admin;
GRANT SELECT ON public.users TO migration_admin;

- grant permissions to the pglogical schema and tables for the gmemegen_db database
\c gmemegen_db;

GRANT USAGE ON SCHEMA pglogical TO migration_admin;
GRANT ALL ON SCHEMA pglogical TO migration_admin;

GRANT SELECT ON pglogical.tables TO migration_admin;
GRANT SELECT ON pglogical.depend TO migration_admin;
GRANT SELECT ON pglogical.local_node TO migration_admin;
GRANT SELECT ON pglogical.local_sync_status TO migration_admin;
GRANT SELECT ON pglogical.node TO migration_admin;
GRANT SELECT ON pglogical.node_interface TO migration_admin;
GRANT SELECT ON pglogical.queue TO migration_admin;
GRANT SELECT ON pglogical.replication_set TO migration_admin;
GRANT SELECT ON pglogical.replication_set_seq TO migration_admin;
GRANT SELECT ON pglogical.replication_set_table TO migration_admin;
GRANT SELECT ON pglogical.sequence_state TO migration_admin;
GRANT SELECT ON pglogical.subscription TO migration_admin;

- grant permissions to the public schema and tables for the gmemegen_db database
GRANT USAGE ON SCHEMA public TO migration_admin;
GRANT ALL ON SCHEMA public TO migration_admin;

GRANT SELECT ON public.meme TO migration_admin;

- Make the migration_admin user the owner of the tables in the orders database
\c orders;
\dt

ALTER TABLE public.distribution_centers OWNER TO migration_admin;
ALTER TABLE public.inventory_items OWNER TO migration_admin;
ALTER TABLE public.order_items OWNER TO migration_admin;
ALTER TABLE public.products OWNER TO migration_admin;
ALTER TABLE public.users OWNER TO migration_admin;
\dt

\q

exit

### Create a Database Migration Service connection profile for a stand-alone PostgreSQL database
- Get the connectivity information for the PostgreSQL source instance
Compute Engine > VM instances > postgresql-vm > Internal IP
- Create a new connection profile for the PostgreSQL source instance
Database Migration > Connection profiles > Create Profile
Database engine = PostgreSQL
Connection profile name = enter postgres-vm
Hostname or IP address = enter the internal IP for the PostgreSQL
Port = enter 5432
Username = migration_admin
Password = DMS_1s_cool! 
Region select (region)
For all other values leave the defaults.
Create

### Create and start a continuous migration job
Database Migration > Migration jobs > Create Migration Job.
Migration job name = vm-to-cloudsql.
Source database engine =  PostgreSQL.
Destination region = (region).
Destination database engine = Cloud SQL for PostgreSQL.
Migration job type = Continuous
Leave the defaults for the other settings
Save & Continue
- Define the source instance
Source connection profile ໍ postgres-vm.
Leave the defaults for the other settings.
Save & Continue
- Create the destination instance
### Allow access to the postgresql-vm instance from automatically allocated IP range
VPC network > VPC network peering > servicenetworking-googleapis-com [Imported routes]
sudo nano /etc/postgresql/13/main/pg_hba.conf
```
# replace using above ip
#GSP918 - allow access to all hosts
host    all all 0.0.0.0/0   md5
```
sudo systemctl start postgresql@13-main
Database Migration Service > Test Job

After a successful test, click Create & Start Job
### Confirm the data in Cloud SQL for PostgreSQL
- Review the data in the Cloud SQL for PostgreSQL instance
\c orders;
select * from distribution_centers;
\q
- Update stand-alone source data to test continuous migration
export VM_NAME=postgresql-vm
export PROJECT_ID=$(gcloud config list --format 'value(core.project)')
export POSTGRESQL_IP=$(gcloud compute instances describe ${VM_NAME} --zone=us-central1-c --format="value(networkInterfaces[0].accessConfigs[0].natIP)")
echo $POSTGRESQL_IP
psql -h $POSTGRESQL_IP -p 5432 -d orders -U migration_admin

### Promote Cloud SQL to be a stand-alone instance for reading and writing data
Database Migration > Migration jobs > vm-to-cloudsql  > Promote > Promote

## Connect an App to a Cloud SQL for PostgreSQL Instance [GSP919]
### initialize apis and create a cloud iam service account
- Enable the API
gcloud services enable artifactregistry.googleapis.com
- Create a Service Account for Cloud SQL
export PROJECT_ID=$(gcloud config list --format 'value(core.project)')
export CLOUDSQL_SERVICE_ACCOUNT=cloudsql-service-account

gcloud iam service-accounts create $CLOUDSQL_SERVICE_ACCOUNT --project=$PROJECT_ID

gcloud projects add-iam-policy-binding $PROJECT_ID --member="serviceAccount:$CLOUDSQL_SERVICE_ACCOUNT@$PROJECT_ID.iam.gserviceaccount.com" --role="roles/cloudsql.admin" 

- export keys to a local file
gcloud iam service-accounts keys create $CLOUDSQL_SERVICE_ACCOUNT.json --iam-account=$CLOUDSQL_SERVICE_ACCOUNT@$PROJECT_ID.iam.gserviceaccount.com --project=$PROJECT_ID

### deploy a lightweight GKE application
- Create a Kubernetes cluster
ZONE=us-central1-c
gcloud container clusters create postgres-cluster --zone=$ZONE --num-nodes=2
- Create Kubernetes secrets for database access
kubectl create secret generic cloudsql-instance-credentials --from-file=credentials.json=$CLOUDSQL_SERVICE_ACCOUNT.json
    
kubectl create secret generic cloudsql-db-credentials --from-literal=username=postgres --from-literal=password=supersecret! --from-literal=dbname=gmemegen_db
- Download and build the GKE application container
gsutil -m cp -r gs://spls/gsp919/gmemegen .
cd gmemegen
- Create environment variables for the region, Project ID and Artifact Registry repository
export REGION=us-central1
export PROJECT_ID=$(gcloud config list --format 'value(core.project)')
export REPO=gmemegen
- Configure Docker authentication for the Artifact Registry
gcloud auth configure-docker ${REGION}-docker.pkg.dev
- Create the Artifact Registry repository
gcloud artifacts repositories create $REPO --repository-format=docker --location=$REGION
- Build a local Docker image
docker build -t ${REGION}-docker.pkg.dev/${PROJECT_ID}/gmemegen/gmemegen-app:v1 .
- Push the image to the Artifact Registry
docker push ${REGION}-docker.pkg.dev/${PROJECT_ID}/gmemegen/gmemegen-app:v1
- Configure and deploy the GKE application
kubectl create -f gmemegen_deployment.yaml
kubectl get pods

### connect GKE application to an external load balancer
- Create a load balancer to make your GKE application accessible from the web
kubectl expose deployment gmemegen --type "LoadBalancer" --port 80 --target-port 8080
- Test the application to generate some data
kubectl describe service gmemegen
- create a clickable link to the external IP address of the load balancer
export LOAD_BALANCER_IP=$(kubectl get svc gmemegen -o=jsonpath='{.status.loadBalancer.ingress[0].ip}' -n default)
echo gMemegen Load Balancer Ingress IP: http://$LOAD_BALANCER_IP
- view the application’s activity
POD_NAME=$(kubectl get pods --output=json | jq -r ".items[0].metadata.name")
kubectl logs $POD_NAME gmemegen | grep "INFO"
- 
### verify full read/write capabilities of application to database
- Connect to the database and query an application table
Databases > SQL > postgres-gmemegen > Overview > Connect to this instance > Open Cloud Shell
\c gmemegen_db
select * from meme;

## securing a cloud sql for postgresql instance

## configure replication and enable point-in-time-recovery for cloud sql for postgresql

## create and manage cloud sql for postgresql instances: challenge lab
