# Security Best Practices in Google Cloud

## securing compute engine: techniques and best practices
### Configuring, Using, and Auditing VM Service Accounts and Scopes
- Create and manage service accounts
gcloud iam service-accounts create my-sa-123 --display-name "my service account"
gcloud projects add-iam-policy-binding $DEVSHELL_PROJECT_ID --member serviceAccount:my-sa-123@$DEVSHELL_PROJECT_ID.iam.gserviceaccount.com --role roles/editor
- Use the client libraries to access BigQuery from a service account
IAM & Admin > Service accounts > Create service account
Create and Continue > BigQuery > BigQuery Data Viewer > Add Another Role > BigQuery > BigQuery User > Continue > Done
- create vm
Compute Engine > VM Instances > Create instance
Security > Turn on Secure Boot > Create
sudo apt-get update -y
sudo apt-get install -y git python3-pip
sudo pip3 install six==1.13.0
sudo pip3 install --upgrade pip
sudo pip3 install --upgrade google-cloud-bigquery
sudo pip3 install pandas
echo "
from google.auth import compute_engine
from google.cloud import bigquery

credentials = compute_engine.Credentials(
    service_account_email='YOUR_SERVICE_ACCOUNT')

query = '''
SELECT
  year,
  COUNT(1) as num_babies
FROM
  publicdata.samples.natality
WHERE
  year > 2000
GROUP BY
  year
'''

client = bigquery.Client(
    project='YOUR_PROJECT_ID',
    credentials=credentials)
print(client.query(query).to_dataframe())
" > query.py
sed -i -e "s/YOUR_PROJECT_ID/$(gcloud config get-value project)/g" query.py
sed -i -e "s/YOUR_SERVICE_ACCOUNT/bigquery-qwiklab@$(gcloud config get-value project).iam.gserviceaccount.com/g" query.py
sudo pip3 install pyarrow==5.0.0
sudo pip3 install db-dtypes
python3 query.py
## securing cloud data: techniques and best practices
### Creating a BigQuery Authorized View
- create the source dataset

- create the analyst dataset

- secure the analyst dataset

- secure the source dataset

- test your security settings

## application security: techniques and best practices
### Identify Application Vulnerabilities with Security Command Center
- launch a virtual machine and deploy a vulnerable application

- scan the application with web security scanner

- correct the vulnerability and scan again

### Securing Compute Engine Applications with BeyondCorp Enterprise
- create a compute engine template

- create a managed instance group

- create a google cloud self-managed SSL certificate resource

- create a load balancer

- restart you VMs

- set up IAP

- test IAP

### Configuring and Using Credentials with Secret Manager
- enable the secret manager api

- create a secret

- use a secret

- create and use a new secret version

- create a new secret version

- reinstate and verify a previous secret version

## securing google kubernetes engine: techniques and best practices
- 