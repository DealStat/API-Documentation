# DealStat API Documentation


DealStat’s API allows integration partners to create new deals for DealStat users, update deals by providing a PDF property offering memos to be read by DealStat’s AI document processor, as well as retrieve and make manual updates to deal data points.

This document provides user-friendly examples for running a typical workflow with DealStat's API, and additional details on the Deal Data Point schema.  For a more formal definition of each available route, visit out [Swagger documentation](https://app.swaggerhub.com/apis/dealstat/DealStat/1.0.2).

The example code included below allows you to:<br />
1. Authenticate an existing DealStat user.<br />
2. Create a new deal for that user.<br />
3. Post a PDF source document to that deal to be read by DealStat's AI document processor.<br />
4. Check the status of the source document sent to the document processor.<br />
5. Retrieve deal data points for a deal.<br />
6. Update the sample deal's acquisition price manually.<br />


## Preparing Your First API Call

The example code below uses **Python 3.6** and the python *requests* and *json* libraries.

In order to run this sample code, you will need to have the following:  
- YOUR_DEALSTAT_API_KEY: Your API key provided by DealStat.
- DEALSTAT_USER_EMAIL: The email address for a registered DealStat user.
- DS_USER_PASSWORD: The user's password.
- TEST_PDF_URL: An address of a PDF document to be processed. 

<br />
	
First, import the libraries needed to send and receive JSON information via the API.

	import requests, json

<br />

## Authenticate the DealStat User

DealStat is built around users and deals.  Each deal is assigned to a user.  In order to create or edit a deal, you need to authenticate the DealStat user to be associated with that deal.

### Sample Request

To authenticate a user, send a **POST** request to the **/authenticate** route.
	
	# Define headers to be sent to API.
	headers = {'content-type':'application/json', 'DealStatAPIKey':'<YOUR_DEALSTAT_API_KEY>'}

	# Define data to be sent to API to authenticate a DealStat user and format as a string.
	# Note that this is a temporary schema to be used with DealStat test accounts.  
    data = {'email':'<DEALSTAT_USER_EMAIL>', 'password':'<DS_USER_PASSWORD>'}
	data = json.dumps(data)	

	# Use requests library to call DealStat API and format response into JSON.
	r = requests.post('http://api.dealstat.co/authenticate', data=data, headers=headers)
	response = json.loads(r.text)
	print(response)

<br />
 
### Sample Response 


	response = {
	    "user_access_token": "USER_ACCESS_TOKEN"
	}

The user_access_token is required for any calls associated with that user and its deals.  It expires after 24 hours of non-use.

<br />

## Create a New Deal for a User

Before doing anything with a deal, you'll need to create one.

### Samples Request 

To create a new deal, send a **POST** request to the **/deal** route.

Note the addition of the user access token (response['user_access_token'] from the previous step) to the 'UserAuthorization' key in the headers.  This tells DealStat which user to associate the new deal with.

	# Define headers.
	headers = {'content-type':'application/json', 'DealStatAPIKey':'<YOUR_DEALSTAT_API_KEY>', 'UserAuthorization':'<USER_ACCESS_TOKEN>'}

	# Use requests library to call DealStat API and format response into JSON.
	r = requests.post('http://api.dealstat.co/deal', headers=headers)
	response = json.loads(r.text)
	print(response)

<br />

### Sample Response 


	response = {
	    "deal_id": "DEAL_ID",
	    "code": "success"
	}

The **response['deal_id']** will be required for any actions associated with that deal.

<br />


## Add a Source Document to a Deal

### Sample Request 

To send a PDF offering memo to be processed by DealStat's A.I. document processor, send a **POST** request to the **/deal/<DEAL_ID>/source_documents** route.

You'll need to define the deal ID (response['deal_id'] from the prior step) within the route, pass a URL for a PDF offering memo, and define the document type ('pdf' | 'listing_link').  In this case, the document type is a PDF. 

	# Define headers and data needed to sent source document.
	headers = {'content-type':'application/json', 'DealStatAPIKey':'<YOUR_DEALSTAT_API_KEY>', 'UserAuthorization':'<USER_ACCESS_TOKEN>'}
	data = {'document_url': '<OFFERING_MEMO_URL>', 'document_type': 'pdf'}
	data = json.dumps(data)	

	# Use requests library to call DealStat API and format response into JSON.
	r = requests.post('http://api.dealstat.co/deal/<DEAL_ID>/source_documents', data=data, headers=headers)
	response = json.loads(r.text)
	print(response)

<br />
 
### Sample Response
	
	response = {
	    "source_document_id": "<SOURCE_DOCUMENT_ID>",
	    "code": "success"
	}

A success message indicates that the document has reached DealStat's document processor.  Depending on the type and size of document, DealStat's machine learning models can take up to a couple of minutes to process a document.
The **response['source_document_id']** can be used to check the status of the document.

<br />


## Check a Source Document Status

### Sample Request 

To check that status of a deal's source documents, send a **GET** request to the **/deal/<DEAL_ID>/source_documents** route.

Depending on the type and size of document, DealStat's machine learning models can take up to a couple of minutes to process a document. 

**Note: To avoid IP address and API key restrictions, wait at least 3 seconds between each API request if checking a source document's status immediately after upload.**

	headers = {'content-type':'application/json', 'DealStatAPIKey':'<YOUR_DEALSTAT_API_KEY>', 'UserAuthorization':'<USER_ACCESS_TOKEN>'}

	r = requests.get('http://api.dealstat.co/deal/<DEAL_ID>/source_documents', headers=headers)
	response = json.loads(r.text)
	print(response)

<br />

### Sample Response
	
	response = {
	    "source_document_info": [
	        {
	            "source_document_id": "<SOURCE_DOCUMENT_ID>",
        	    "source_document_type": "pdf",
	            "processing_status": True
	        }
	    ],
	    "code": "success"
	}	

This route returns a list of dictionaries corresponding to each source document associated with the deal, which includes the document's type and processing status.  If the processing status is *true*, any data extracted from that source document has been included in the deal's data points.

<br />



## Request Deal Data Points

### Sample Request

To request all data points associated with a deal, send a **GET** request to the **/deal/<DEAL_ID>/data_points** route.

	headers = {'content-type':'application/json', 'DealStatAPIKey':'<YOUR_DEALSTAT_API_KEY>', 'UserAuthorization':'<USER_ACCESS_TOKEN>'}

	r = requests.get('http://api.dealstat.co/deal/<DEAL_ID>/data_points', headers=headers)
	response = json.loads(r.text)
	print(response)


<br /> 


### Sample Response

A subset of a sample response['data_points'] from the API: 
	
	[
	...
	{
	'key': 'acquisition_price'	
	'value': {
		'amount': 4800000.0, 
		'period': 1, 
		'evaluated': 4800000.0, 
		'area': 'total'
		}, 
	'source': {
		'source_document_type': 'pdf', 
		'user_id': None, 
		'verbose': '', 
		'calculated': False, 
		'derived': False, 
		'calculated_from': [], 
		'table_number': 0, 
		'page_number': '6', 
		'derived_from': [], 
		'listing_link': None, 
		'source_id': '<SOURCE_DOCUMNET_ID>', 
		'pdf_sub': 'table', 
		'main': 'source_document'
		}, 	
	},
	...
	]

<br />

*response['data_points']* will contain a list of dictionaries.  Each dictionary represents a data point, and contains: 

- key 
- value 
- source 

<br />

#### Data Point Key 

The *key* represents the name of the data point (i.e. 'acquisition_price' or 'net_operating_income').

<br /> 

#### Data Point Value 

The *value* represents the value of that data point.  This is broken into multiple sub-keys to provide context and modifiers.

| Value Sub-key       | Expected Values     | Description           | 
| ------------------ | ------------------- | --------------------- |
| amount             | Depends on value type. | Raw value (i.e. 1000000 or 0.087).             | 
| period             | 1 \| 12 | Number of occruances per year (i.e. 1 to represent annually; 12 to represent monthly).  Defaults to 1 for N/A values.      | 
| area               | 'total' \| 'PSF' \| 'per_unit' \| 'cap_rate' (others...) | Qualifier such as 'PSF', 'per_unit', or 'cap_rate'.  Defaults to 'total'.      | 
| evaluated          | *float* | Specifically used for 'acquisition_price' and 'exit_price', this represents the evaluated whole dollar amount (i.e. the amount may be a cap rate or PPSF, but the 'evaluated' amount represents the dollar value after accounting for the NOI or SF, respectively.      | 

<br />

#### Data Point Source 

This provides detailed information on the source for the data point.


| Value Sub-key            | Expected Values       | Description           |
| ----------------------- | --------------------- | --------------------- |
| main  | 'source_document' \| 'manual' \| 'financial_model' \| 'api'  | Indicates the primary source of the data point. |
| source_document_type  | 'pdf' \| 'listing_link'   | Type of source document this data point was extracted from. |
| source_id  | *string*   | Unique ID for source document. |
| listing_link  | *string (url)*   | URL for listing link of source. |
| calculated  | *boolean*   | Indicates whether value was calculated directly from other values. |
| calculated_from  | *list*   | List of data point keys used the calculate value. |
| derived  | *boolean*   | Indicates whether value was derived directly from other values by implication. |
| derived_from  | *list*    | List of data point keys used the derive value. |
| page_number  | *string*   | Page number within PDF document page. |
| pdf_sub  |  'table' \| 'text'  | Indicates whether data point from PDF comes from a table or document text. |
| table_number  |  *string*  | Table number within PDF document page.  |
 
<br /> 



## Update a Deal Data Point

### Sample Request 

To update a deal data point, send a **POST** request to the **/deal/<DEAL_ID>/data_points** route.

In this example, we will update the 'acquisition_price' to an amount of $4,000,000, an area of 'total'.

	headers = {'content-type':'application/json', 'DealStatAPIKey':'<YOUR_DEALSTAT_API_KEY>'}
	
	data = {
	  'updates': [
	    {
	      'key': 'acquisition_price',
	      'value': {'amount':4000000, 'area':'total'}
	    }
	  ]
	}
	data = json.dumps(data)	

	r = requests.post('http://api.dealstat.co/deal/<DEAL_ID>/data_points', data=data, headers=headers)
	response = json.loads(r.text)
	print(response)

<br />

Each key has a certain required schema to be used when updating.  (More detail coming soon...)  

<br />



### Sample Response
	
	response = {
		'code': 'success'
	}


A success message indicates that the value has been updated.  You can repeat the GET request to /deal/<DEAL_ID>/data_points to view the updated value.

<br />

