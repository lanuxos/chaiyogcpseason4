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
## Migrating to AlloyDB from PostgreSQL Using Database Migration Service [GSP1084]
student-00-eb12a87b4f91@qwiklabs.net
jLGMXHFz7ySX
qwiklabs-gcp-03-eef20b05f8a4
us-east4
us-east4-b
### verify data in the source instance for migration
sudo -u postgres psql
\dt
select count (*) as countries_row_count from countries;
select count (*) as departments_row_count from departments;
select count (*) as employees_row_count from employees;
select count (*) as jobs_row_count from jobs;
select count (*) as locations_row_count from locations;
select count (*) as regions_row_count from regions;
\q
exit
### create a database migration service connection profile for a stand-alone postgresql database
Database Migration > Connection profiles.
Create Profile
Source database engine : PostgreSQL
Connection profile : pg14-source
IP
Port : 5432
Username : postgres
Password : Change3Me 
Region : us-east4
Create
### create and start a continuous migration job
- Create a new continuous migration job
Database Migration > Migration jobs
Create Migration Job
name : postgres-to-alloydb
Source database engine : PostgreSQL
Destination database engine : AlloyDB PostgreSQL
Destination region : us-east4
Migration job type: Continuous
Save & Continue
- Define the source instance
Source connection profile : pg14-source
Save & Continue
- Create the destination instance
Cluster Type : Highly available
Continue
Cluster ID : alloydb-target-cluster
Password : Change3Me
Network, select peering-network
Continue
Instance ID : alloydb-target-instance
Select 2 vCPU, 16 GB as your machine type
Save & Continue
Create Destination & Continue to continue.
Connectivity method : VPC peering
Configure & Continue
Configure & Continue
- Test and start the continuous migration job

### confirm data load in the alloyDB for postgresql instance
- ssh into destination instance [alloyDB]
psql -h $ALLOYDB -U postgres
\dt

select count (*) as countries_row_count from countries;
select count (*) as departments_row_count from departments;
select count (*) as employees_row_count from employees;
select count (*) as jobs_row_count from jobs;
select count (*) as locations_row_count from locations;
select count (*) as regions_row_count from regions;

select region_id, region_name from regions;
### propagate a alive update to the alloyDB instance
- ssh into source instance and add record
sudo -u postgres psql
insert into regions values (5, 'Oceania');
select region_id, region_name from regions;
- check destination instance for changes
select region_id, region_name from regions;
## Migrating to Alloy DB from PostgreSQL Using PostgreSQL Tools
### 
## Administering an AlloyDB Database
### 
## Accelerating Analytical Queries using the AlloyDB Columnar Engine
### 
## Create and Manage AlloyDB Instances: Challenge Lab
### 