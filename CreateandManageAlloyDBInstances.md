# Create and Manage AlloyDB Instances
## AlloyDB - Database Fundamentals [GSP1083]
student-03-a5e6be22ac54@qwiklabs.net
hAvccbc5YVsH
qwiklabs-gcp-04-b396c7c152f1
us-east4
us-east4-c
10.40.0.2:5432
### create a cluster and instance
Databases  > AlloyDB for PostgreSQL > Clusters > Create cluster > Highly Available > Continue
Cluster ID : lab-cluster
Password : Change3Me
Network : peering-network
Continue
machine type : 2 vCPU, 16GB
Create Cluster
### create tables and insert data in your database
Compute Engine > VM instances > alloydb-client > Connect [SSH]
export ALLOYDB=Instance_IP
- launch PostgreSQL
psql -h $ALLOYDB -U postgres
- create table
CREATE TABLE regions (
    region_id bigint NOT NULL,
    region_name varchar(25)
) ;

ALTER TABLE regions ADD PRIMARY KEY (region_id);
- insert records into table
INSERT INTO regions VALUES ( 1, 'Europe' );

INSERT INTO regions VALUES ( 2, 'Americas' );

INSERT INTO regions VALUES ( 3, 'Asia' );

INSERT INTO regions VALUES ( 4, 'Middle East and Africa' );
- query
SELECT region_id, region_name from regions;
\q
### use the google cloud cli with alloyDB
- create cluster
gcloud beta alloydb clusters create gcloud-lab-cluster --password=Change3Me --network=peering-network --region=us-east4 --project=qwiklabs-gcp-04-b396c7c152f1
- create instance
gcloud beta alloydb instances create gcloud-lab-instance --instance-type=PRIMARY --cpu-count=2 --region=us-east4  --cluster=gcloud-lab-cluster --project=qwiklabs-gcp-04-b396c7c152f1
- list instances
gcloud beta alloydb clusters list
### deleting a cluster
gcloud beta alloydb clusters delete gcloud-lab-cluster --force --region=us-east4 --project=qwiklabs-gcp-04-b396c7c152f1
gcloud beta alloydb clusters list
## Migrating to AlloyDB from PostgreSQL Using Database Migration Service
### 
## Migrating to Alloy DB from PostgreSQL Using PostgreSQL Tools
### 
## Administering an AlloyDB Database
### 
## Accelerating Analytical Queries using the AlloyDB Columnar Engine
### 
## Create and Manage AlloyDB Instances: Challenge Lab
### 