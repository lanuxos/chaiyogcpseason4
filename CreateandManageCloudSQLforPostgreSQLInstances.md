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

## securing a cloud sql for postgresql instance [GSP920]
### create a cloud sql for postgresql instance with cmek enabled
- Create a per-product, per-project service account for Cloud SQL
export PROJECT_ID=$(gcloud config list --format 'value(core.project)')
gcloud beta services identity create --service=sqladmin.googleapis.com --project=$PROJECT_ID
- Create a Cloud Key Management Service keyring and key
export KMS_KEYRING_ID=cloud-sql-keyring
export ZONE=$(gcloud compute instances list --filter="NAME=bastion-vm" --format=json | jq -r .[].zone | awk -F "/zones/" '{print $NF}')
export REGION=${ZONE::-2}
gcloud kms keyrings create $KMS_KEYRING_ID --location=$REGION
- create the Cloud KMS key
export KMS_KEY_ID=cloud-sql-key
gcloud kms keys create $KMS_KEY_ID --location=$REGION --keyring=$KMS_KEYRING_ID --purpose=encryption
- bind the key to the service account
export PROJECT_NUMBER=$(gcloud projects describe ${PROJECT_ID} --format 'value(projectNumber)')
gcloud kms keys add-iam-policy-binding $KMS_KEY_ID --location=$REGION --keyring=$KMS_KEYRING_ID --member=serviceAccount:service-${PROJECT_NUMBER}@gcp-sa-cloud-sql.iam.gserviceaccount.com --role=roles/cloudkms.cryptoKeyEncrypterDecrypter
- Create a Cloud SQL instance with CMEK enabled
- find the external IP address of the bastion-vm VM instance
export AUTHORIZED_IP=$(gcloud compute instances describe bastion-vm --zone=$ZONE --format 'value(networkInterfaces[0].accessConfigs.natIP)')
echo Authorized IP: $AUTHORIZED_IP
34.168.202.26
- find the external IP address of the Cloud Shell
export CLOUD_SHELL_IP=$(curl ifconfig.me)
echo Cloud Shell IP: $CLOUD_SHELL_IP
35.198.215.15
- create your Cloud SQL for PostgreSQL instance
export KEY_NAME=$(gcloud kms keys describe $KMS_KEY_ID --keyring=$KMS_KEYRING_ID --location=$REGION --format 'value(name)')

export CLOUDSQL_INSTANCE=postgres-orders
gcloud sql instances create $CLOUDSQL_INSTANCE --project=$PROJECT_ID --authorized-networks=${AUTHORIZED_IP}/32,$CLOUD_SHELL_IP/32 --disk-encryption-key=$KEY_NAME --database-version=POSTGRES_13 --cpu=1 --memory=3840MB --region=$REGION --root-password=supersecret!
- 

### enable and configure pgaudit on a cloud sql for postgresql database
- add the pgAudit database flags
gcloud sql instances patch $CLOUDSQL_INSTANCE --database-flags cloudsql.enable_pgaudit=on,pgaudit.log=all
Databases > SQL > postgres-orders > Overview > Restart > Restart
Connect to this instance > Open Cloud Shell
CREATE DATABASE orders;
\c orders;
CREATE EXTENSION pgaudit;
ALTER DATABASE orders SET pgaudit.log = 'read,write';
- Enable Audit Logging in Cloud Console
IAM & Admin > Audit Logs > Filter > Cloud SQL > Info Panel > Save
- Populate a database on Cloud SQL for PostgreSQL
export SOURCE_BUCKET=gs://cloud-training/gsp920
gsutil -m cp ${SOURCE_BUCKET}/create_orders_db.sql .
gsutil -m cp ${SOURCE_BUCKET}/DDL/distribution_centers_data.csv .
gsutil -m cp ${SOURCE_BUCKET}/DDL/inventory_items_data.csv .
gsutil -m cp ${SOURCE_BUCKET}/DDL/order_items_data.csv .
gsutil -m cp ${SOURCE_BUCKET}/DDL/products_data.csv .
gsutil -m cp ${SOURCE_BUCKET}/DDL/users_data.csv .
- create and populate the database
export CLOUDSQL_INSTANCE=postgres-orders
export POSTGRESQL_IP=$(gcloud sql instances describe $CLOUDSQL_INSTANCE --format="value(ipAddresses[0].ipAddress)")
export PGPASSWORD=supersecret!
psql "sslmode=disable user=postgres hostaddr=${POSTGRESQL_IP}" -c "\i create_orders_db.sql"
exit
CREATE ROLE auditor WITH NOLOGIN;
ALTER DATABASE orders SET pgaudit.role = 'auditor';
GRANT SELECT ON order_items TO auditor;
SELECT
    users.id  AS users_id,
    users.first_name  AS users_first_name,
    users.last_name  AS users_last_name,
    COUNT(DISTINCT order_items.order_id ) AS order_items_order_count,
    COALESCE(SUM(order_items.sale_price ), 0) AS order_items_total_revenue
FROM order_items
LEFT JOIN users ON order_items.user_id = users.id
GROUP BY 1, 2, 3
ORDER BY 4 DESC
LIMIT 500;
SELECT
    products.id  AS products_id,
    products.name  AS products_name,
    products.sku  AS products_sku,
    products.cost  AS products_cost,
    products.retail_price  AS products_retail_price,
    products.distribution_center_id  AS products_distribution_center_id,
    COUNT(DISTINCT order_items.order_id ) AS order_items_order_count,
    COALESCE(SUM(order_items.sale_price ), 0) AS order_items_total_revenue
FROM order_items
LEFT JOIN inventory_items ON order_items.inventory_item_id = inventory_items.id
LEFT JOIN products ON inventory_items.product_id = products.id
GROUP BY 1, 2, 3, 4, 5, 6
ORDER BY 7 DESC
LIMIT 500;
SELECT
    order_items.order_id AS order_id,
    distribution_centers.id  AS distribution_centers_id,
    distribution_centers.name  AS distribution_centers_name,
    distribution_centers.latitude  AS distribution_centers_latitude,
    distribution_centers.longitude  AS distribution_centers_longitude
FROM order_items
LEFT JOIN inventory_items ON order_items.inventory_item_id = inventory_items.id
LEFT JOIN products ON inventory_items.product_id = products.id
LEFT JOIN distribution_centers ON products.distribution_center_id = distribution_centers.id
GROUP BY 1, 2, 3, 4, 5
ORDER BY 2
LIMIT 500;
\q
- View pgAudit logs
Observablity > Logging > Logs Explorer > Query > Logs Explorer > Run query
resource.type="cloudsql_database"
logName="projects/(GCP Project)/logs/cloudaudit.googleapis.com%2Fdata_access"
protoPayload.request.@type="type.googleapis.com/google.cloud.sql.audit.v1.PgAuditEntry"
### configure cloud sql iam database authentication
- Test database access using a Cloud IAM user before Cloud SQL IAM authentication is configured
export USERNAME=$(gcloud config list --format="value(core.account)")
export CLOUDSQL_INSTANCE=postgres-orders
export POSTGRESQL_IP=$(gcloud sql instances describe $CLOUDSQL_INSTANCE --format="value(ipAddresses[0].ipAddress)")
export PGPASSWORD=$(gcloud auth print-access-token)
psql --host=$POSTGRESQL_IP $USERNAME --dbname=orders
- Create a Cloud SQL IAM user
Databases > SQL > Overview > Configuration > Database flags > pgAudit.log > cloudsql.enable_pgaudit > Users > Users > Add user account > Cloud IAM > Principal > Add > Overview > cloudsql.iam_authentication
- Grant the Cloud IAM user access to a Cloud SQL database table
gcloud sql connect postgres-orders --user=postgres --quiet
\c orders
GRANT ALL PRIVILEGES ON TABLE order_items TO "[IAM Username]";
\q
- Test database access using a Cloud IAM user after Cloud SQL IAM authentication is configured
export PGPASSWORD=$(gcloud auth print-access-token)
psql --host=$POSTGRESQL_IP $USERNAME --dbname=orders
SELECT COUNT(*) FROM order_items;
SELECT COUNT(*) FROM users;
## configure replication and enable point-in-time-recovery for cloud sql for postgresql [GSP922]
### enable backups on the cloud sql for postgresql instance
- display the instance details
export CLOUD_SQL_INSTANCE=postgres-orders
gcloud sql instances describe $CLOUD_SQL_INSTANCE
- get current UTC time
date +"%R"
- enable scheduled back-ups [HH:MM use earlier step ouput]
gcloud sql instances patch $CLOUD_SQL_INSTANCE     --backup-start-time=HH:MM
- confirm your changes
gcloud sql instances describe $CLOUD_SQL_INSTANCE --format 'value(settings.backupConfiguration)'
### enable and run point-in-time recovery
- enable point-in-time-recovery
  gcloud sql instances patch $CLOUD_SQL_INSTANCE enable-point-in-time-recovery retained-transaction-log-days=1
- make change to database
Databases > SQL > postgres-orders > Cloud Shell
\c orders;
select count(*) from distribution_centers;
start new cloud shell and type to get timestamp for further step
date --rfc-3339=seconds
INSERT INTO distribution_centers VALUES(-80.1918,25.7617,'Miami FL',11);
SELECT COUNT(*) FROM distribution_centers;
\q
- perform a point-in-time recovery
export NEW_INSTANCE_NAME=postgres-orders-pitr
gcloud sql instances clone $CLOUD_SQL_INSTANCE $NEW_INSTANCE_NAME --point-in-time 'TIMESTAMP'
### confirm database has been restored to the correct point-int-time
- connect to instance
gcloud sql connect postgres-orders --user=postgres --quiet
- use database
\c orders;
- count
SELECT COUNT(*) FROM distribution_centers;
## create and manage cloud sql for postgresql instances: challenge lab
### migrate a stand-alone postgresql database to a cloud sql for postgresql instance
Migration user name : Postgres Migration User
Migration user password : DMS_1s_cool!
- Prepare the stand-alone PostgreSQL database for migration
- Migrate the stand-alone PostgreSQL database to a Cloud SQL for PostgreSQL instance
- 
### promote a cloud sql to be a stand-alone instance for reading and writing data
- 
### implement cloud sql for postgresql iam database authentication
- 
### configure and test point-in-time recovery
- 