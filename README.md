# Searchium.ai™

# OpenSearch/Elasticsearch Plugin

## Documentation


## Contents

- 1. Introduction
- 2. Setting Up OpenSearch
   - 2.1.Configuring a VPC
   - 2.2.Configuring the Transit Gateway
   - 2.3.Setting Up the Plugin
      - 2.3.1. Registering Your IP Address
      - 2.3.2. Configuring an EC2
      - 2.3.3. Configuring Server Locations
      - 2.3.4. Specifying Dataset Location
   - 2.4.Associating a Cloud Service
- 3. Indexing a Dataset
- 4. Working with the Plugin
   - 4.1.Logging In
   - 4.2.Managing Search Data
      - 4.2.1. Viewing Available Datasets
      - 4.2.2. Uploading and Training Datasets
      - 4.2.3. Loading Trained Datasets Onto the APU
      - 4.2.4. Deleting an Indexed Dataset
   - 4.3.Querying a Dataset
      - 4.3.1. Specifying the Top-k and Indices
      - 4.3.2. Performing Queries


## 1. Introduction

The Searchium.ai OpenSearch/Elasticsearch plugin provides a convenient way to
run OpenSearch/Elasticsearch queries using Searchium.ai APU servers over the
cloud.

## 2. Setting Up OpenSearch

Some setup is required before using OpenSearch for the first time, including:

- Configuring a Virtual Private Cloud (VPC)
- Configuring your transit gateway
- Setting up the OpenSearch Plugin
- Associating a cloud service
### 2.1.Configuring a VPC

To configure your VPC to work with Searchium.ai:

1. Request an IPv4 CIDR from devopsil@gsitechnology.com. You will receive
    the IPv4 CIDR by email.
2. From your AWS console click VPC. The Resources by Region page appears.
3. Ensure that you are using the US-WEST-1 region.
4. Click VPCs. The Your VPCs page appears.


5. Click Create VPC. The Create VPC page appears.
6. Click Create VPC. The VPC workflow is created.

### 2.2.Configuring the Transit Gateway

To connect to Searchium.ai’s transit gateway:

1. Login to AWS website with your AWS account ID.
2. Open the Resource access manager and select Resource shares from the
    navigation panel on the left.
3. In the Resource shares pane, click the instance to select the invitation.
4. In the shared resource invitation screen, click the Accept resource share
    button.
5. Click OK to confirm acceptance of the resource share.
6. The resource that was shared should be listed in the shared resources table:


7. Select Transit gateway attachments from your virtual private cloud page, and
    then click the Create transit gateway attachment button in the top right corner
    of the panel.
8. In the Create transit gateway attachment screen:


a. In the Details section s elect the transit gateway ID

b. In the VPC attachment section, select the VPC ID of the VPC you
created in 2.1 Configuring a VPC on page 3.

c. Click Create:


9. The transit gateway attachment’s status will change to Pending and then to
    Available:
1. Once the status changes to Available, select VPC > Route tables > Routes>
Edit routes:


2. Click the Add route button.
3. Under Destination, select a relevant IP address. Under Target, select Transit
gateway. Click the Save changes button to confirm the route updates.
### 2.3.Setting Up the Plugin

You can use the Searchium.ai OpenSearch plugin with Searchium.ai’s EC2 or your
own EC2.

Several steps are required before using the plugin for the first time. These include:

- If using Searchium.ai’s EC2 – Registering your IP address with Searchium.ai
- If using your own EC2 – Configuring the EC
- Configuring the plugin with:
    o Server locations
    o Dataset location

#### 2.3.1. Registering Your IP Address

If you are using GSI’s EC2, then you must have access to GSI’s EC2 to use the
plugin. To gain access, send the IP address of your computer to
devopsil@gsitechnology.com.

#### 2.3.2. Configuring an EC2

To configure an EC2 for running the Searchium.ai OpenSearch/Elasticsearch
Plugin:

1. Log in to your AWS account and allocate an EC2 in the VPC you created in 2.
    Configuring a VPC on page 3, with the following properties:
    - m5.10xlarge
    - 40vCPU
    - 32 GB DRAM
    - 256 GB EBS
2. Install OpenSearch 1.2.4/Elasticsearch 7.10.
3. Download the Searchium.ai OpenSearch/Elasticsearch Plugin from the GSI
    GitHub site.

#### 2.3.3. Configuring Server Locations

To configure the Searchium.ai OpenSearch/Elasticsearch Plugin:

1. Update the plugin configuration file with the Searchium.ai server IP.


a. Open opensearch.yml/elasticsearch.yml in a console.

b. Get the IP addresses from the plan’s allocation details section

c. Add the following lines to the file:
gsi.service.host: <your Searchium.ai server IP address>
gsi.search.host: < IP address of OpenSearch server backend>


- **Note:** The IP addresses for gsi.service.host and gsi.search.host should
    be the same.


Example configuration file:
```yaml
cluster.name: "docker-cluster"
network.host: 0.0.0.
path.repo: ["/home/public/elasticsearch/snapshot/my_backup"]
http.cors.enabled: true
http.cors.allow-origin: "*"
http.cors.allow-methods: OPTIONS, HEAD, GET, POST, PUT,DELETE
http.cors.allow-headers: "X-Requested-With,X-Auth-Token,Content-Type, Content-Length, Authorization"
plugins.security.disabled: true
gsi.service.host: <your Searchium.ai server IP address>
gsi.search.host: <IP address of OpenSearch server backend OpenSearch>
```
2. Add the following two lines to the plugin-security.policy file which can be found in
    the plugin zip folder:
    

```yaml
       grant {
       permission java.net.SocketPermission "*",
       "connect,resolve";
       permission java.lang.RuntimePermission "exitVM.0";
       permission java.lang.RuntimePermission
       "accessDeclaredMembers";
       permission java.lang.RuntimePermission "defineClass";
       permission java.lang.reflect.ReflectPermission "suppressAccessChecks";
       permission java.io.FilePermission "/etc/hosts", "read";
       permission java.io.FilePermission "/home/public/elastic-similarity/upload_store/-", "read,write";
       **permission java.io.FilePermission**
       **"/home/<USERNAME>/.aws/credentials", "read";**
       **permission java.io.FilePermission**
       **"/home/<USERNAME>/.aws/config", "read";**
       permission java.io.FilePermission
       "/usr/share/opensearch/.aws/credentials", "read";
       permission java.io.FilePermission
       "/usr/share/opensearch/.aws/config", "read";
       permission java.lang.RuntimePermission
       "accessClassInPackage.sun.misc";
};
```
3. Overwrite the previous opensearch.yml/elasticsearch.yml config file with the
    updated one.
4. Install the Searchium.ai plugin, as you would any OpenSearch plugin.

#### 2.3.4. Specifying Dataset Location

To configure where datasets are stored:

10. Click Plugin settings. The Plugin settings page appears.
11. Click the Advanced settings tab.
12. To store datasets locally:
    a. Click LOCAL FS.
    b. Enter the folder path. If both the plugin and the Searchium.ai backend
       server are installed locally, you can use a local folder. If the plugin and
       backend server are not installed locally, use an EFS shared folder.
13. To store datasets on an AWS S3 server
    a. Click AMAZON S3.
    b. Enter the login information, region, and Amazon bucket name.
3. Click Apply. The file location is saved.


### 2.4.Associating a Cloud Service

A cloud service is an essential part of the Searchium.ai architecture. Associate your
cloud service with your user account before using Searchium.ai service.

To associate your cloud service with Searchium.ai:

1. From the Searchium.ai site, click. The Projects page appears.
2. Click Add cloud service. The Add account window appears.
3. Click Save. The cloud service is associated with your Searchium.ai account.

## 3. Indexing a Dataset

You can index your dataset to use specific fields as filters. To index your dataset:

1. Create a mapping for your index in OpenSearch/Elasticsearch.
    To be used as a filter a field must be:
    - Non-nested object
    - Of type “keyword”
    - A string or a single array of strings
    - Included in all documents
    - Have a non-empty value. Empty values such as “” are not supported.


Examples:

**Valid** filter mappings for "description", "category" and "source"

```json
{
   "settings":{
      "index":{
         "number_of_shards":"1",
         "knn":false,
         "number_of_replicas":"0"
      }
   },
   "mappings":{
      "properties":{
         "description_vector":{
            "type":"knn_vector",
            "dimension":
         },
         "description":{
            "type":"keyword"
         },
         "category":{
            "type":"keyword"
         },
         "source":{
            "type":"text",
            "fields":{
               "keyword":{
                  "type":"keyword",
                  "ignore_above":
               }
            }
         }
      }
   }
}
```


Invalid filter mapping for "category", due to nesting of “properties”.
```json
{
   "mappings":{
      "properties":{
         "description_vector":{
            "type":"knn_vector",
            "dimension":
         },
         "description":{
            "type":"keyword"
         },
         "category":{
            "properties":{
               "sub_category":{
                  "type":"keyword"
               }
            }
         },
         "source":{
            "type":"text",
            "fields":{
               "keyword":{
                  "type":"keyword",
                  "ignore_above":
               }
            }
         }
      }
   }
}
```
**Important:** Your index size must be more than 4, 000 documents.


Indexing example:
```json
{
   "settings":{
      "index":{
         "number_of_shards":"1",
         "knn":false,
         "number_of_replicas":"0"
      }
   },
   "mappings":{
      "properties":{
         "description_vector":{
            "type":"knn_vector",
            "dimension":
         },
         "description":{
            "type":"keyword"
         },
         "sub_category":{
            "type":"keyword"
         },
         "source":{
            "type":"text",
            "fields":{
               "keyword":{
                  "type":"keyword",
                  "ignore_above":
               }
            }
         }
      }
   }
}
```

## 4. Working with the Plugin


The Searchium.ai OpenSearch/Elasticsearch Plugin enables you to:

- Log in
- Manage search data
- Query a dataset

### 4.1.Logging In


To log in:

1. Use < IP address of EC2 >:3080 to open the OpenSearch/Elasticsearch
    Plugin web app. The sign in window appears.
2. Enter the username and password that you received from GSI. The **Login**
    screen appears.


3. Enter the allocation ID from your Searchium.ai cloud management account,
    or that was provided to you by email. The main page appears.

### 4.2.Managing Search Data

The Searchium.ai OpenSearch/Elasticsearch Plugin enables you to work with your
search data, including:

- Viewing available datasets
- Uploading and training datasets
- Loading a dataset onto the APU
- Deleting a dataset

#### 4.2.1. Viewing Available Datasets

The Searchium.ai OpenSearch/Elasticsearch Plugin recognizes
OpenSearch/Elasticsearch datasets that were indexed with vector types of
“ **dense_vector** ” or “ **knn_vector** ”, and field types of “ **keyword** ”.

Dataset documents can include:

- Metadata fields of any OpenSearch/Elasticsearch type.
- FP32 vectors.

To view available datasets:

1. Use < IP address of server backend >:3080 to open the Searchium.ai plugin
    web app.
2. Click Index management. A list of OpenSearch/Elasticsearch datasets that
    were indexed with vector types of “ **dense_vector** ” or “ **knn_vector** ”.appears.


#### 4.2.2. Uploading and Training Datasets

To upload an index:

1. Click the Action icon of an index with Not uploaded status and choose Upload
    from the context menu. The UPLOAD A NEW INDEX page appears.
2. For flat training click FLAT. For cluster training, click CLUSTERS.
3. Click NEXT. The Parameters section appears.


4. Enter the requested parameters. Parameters vary based on training type.
5. Click NEXT. The Pre-filters section appears.
6. In the Prefilter window, choose which filters you would like to use when
    searching through your dataset. Only fields of type “keyword” are available as
    filters.


7. Click NEXT. A summary of your choices appears.
8. Verify your choices.
9. Click SUBMIT. Uploading and training begins.

#### 4.2.3. Loading Trained Datasets Onto the APU

1. Click the Action icon of an index with the Ready to Load status and choose
    Load from the context menu. The ALLOCATIONS window appears.


2. Select an allocation and click Load. The index is loaded and its status
    changes to “Ready to Search”.

#### 4.2.4. Deleting an Indexed Dataset

To remove an index from the Searchium.ai plugins server or to re-upload an index:

1. Click the action icon of an index with the Ready to Load status and choose
    Delete from the context menu. The Delete confirmation window appears.
2. Click OK.


Important: When deleting an index and reindexing it with the same name, wait ~30 seconds
before reindexing. This gives the Searchium.ai backend server time to delete all the old data from the database. When reindexing with a new name, no wait is required.

### 4.3.Querying a Dataset

The Searchium.ai OpenSearch/Elasticsearch Plugin provides tools for:

- Specifying the top-k and indices for query results
- Performing queries

#### 4.3.1. Specifying the Top-k and Indices

Under the General settings tab, enter the Default Top K.

Note: If the “size” value in your search request is less than the Default Top K the “size” value
will be used as the top-k value.

To change the indices available for use, update Managed Indices.

Important: If you make changes to Managed Indices, you will have to upload and train the
dataset again.

#### 4.3.2. Performing Queries

Set up queries, or batches of queries, in the same way that you would set up
queries in OpenSearch/Elasticsearch without the plugin.

Single search query example:

http://HOSTNAME:9200/INDEX_NAME/_search

#####

```json
{
   "query":{
      "gsi_knn":{
         "field":"description_vector",
         "vector":[
            0.0015746655408293009,
            0.025234133005142212,
            0.0031481462065130472....
         ],
         "topk":2,
         "prefilter":{
            "sub_category":"Dress",
            "source":"comsumer"
         },
         "filterOperator":"OR"
      }
   },
   "size":30
}
```

Batch of queries example:

http://HOSTNAME:9200/gsi/search/multiquery/INDEX_NAME/NAME_OF_VECTOR_FIELD

#####

```json
{
   "queries":[
      {
         "vector":[
            0.25973913073539734,
            0.023432284593582153,
            0.07090096920728683,
            0.006839084904640913,
            0.015795692801475525...
         ]
      },
      {
         "vector":[
            0.05060698837041855,
            0,
            0.04455435276031494,
            0.0069176568649709225,
            0.06633245199918747...
         ],
         "prefilter":{
            "description":"Siberian husky",
            "image_url.keyword":"n02096585_6634.JPEG"
         },
         "filterOperator":"OR"
      }
   ],
   "topk":"2"
}
```
D0047 Searchium.ai OpenSearch User Guide Rev1.0






