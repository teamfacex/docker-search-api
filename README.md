[![Licence](https://img.shields.io/cocoapods/l/LivenessSDK?color=red&logo=red)](https://img.shields.io/cocoapods/l/LivenessSDK?color=red&logo=red)

# Face Search Docker

## Introduction
Brought to you by FaceX.io, this docker image can now be used to host FaceX Face Search API on your server.

## Index
* [Setup](#setup)
* [Interacting with the server](#interacting-with-the-server)
  * [Directions](#directions)
* [Add User](#add-user)
  * [Endpoint](#endpoint)  
  * [Response](#repsonse)
* [Delete User](#delete-user)
  * [Endpoint](#endpoint-1)
  * [Response](#response)
* [Build Index](#build-index)
  * [Endpoint](#endpoint-2)
  * [Response](#response-1)
* [Search User](#search-user)
  * [Endpoint](#endpoint-2)
  * [Response](#response-2)
* [Error and Warnings](#error-and-warnings)
  * [Server Side](#server-side-)
  * [API Error Responses](#api-error-responses)
  * [Add User Errors](#add-user-1)
  * [Search User Errors](#search_user)
  * [Delete User Errors](#delete_user)
  

## Setup
1. Download the docker image file provided as a tar archive `face_search.tar.gz`

2. Verify that docker is installed and ready to go.
    ```
    docker --version
    ```
    You can also run a hello-world image to test the installation.
    ```
    docker run hello-world
    ```
3. Load the docker image from the tar archive.
    ```
    docker image load < face_search.tar.gz
    ```
4. Verify that the image is loaded by listing docker images.
    ```
    docker images
    ```
5. Run the docker image
    ```
    docker run -it --rm -p 5000:5000 -e USER_KEY=<license_key> -v ${PWD}/data:/sdk/data facex/search:latest
    ```
    >`-p` &nbsp; : &nbsp; tag chooses the port to serve on  
    > To choose a custom port `xxxx`. Provide the `-p` tag as:  
    > `-p 5000:xxxx`

    >`-v` &nbsp; : &nbsp; tag is used to mount the `data` folder from the 
    >current working directory.  
    > The `data` folder is a protected foler which stores the face_match >database locally.  
    >If the data folder is in a different directory, simply substitue `${PWD}` with the absolute path.
    

6. FaceX API server will start serving on `0.0.0.0:5000`. On a local machine this would be `localhost:5000` or `127.0.0.1:5000`


## Interacting with the server:

You can interact with the server using the exposed rest API.
Documentation for which is provided below.  

Sample codes for Python, can be found in `/sample_codes`

### Directions:
1. Add the users' faces by calling `/add_user` . 
    > Every user entry requires an unique `uid` which is used to identify unique identities.  
    > `name` parameter is used to identify a particular face image.  

    Although a single face image can be provided per user, providing multiple face images per person will favour accuracy.  
    When providing multiple face images, make sure that the `uid` is same for all the face images corresponding to a particular user.  
    \* `name` parameter can be different.  
2. Call `/init` to build the indexes.
    > This is an important step, without performing indexing search is not possible.
3. Call `/search_face` with an unseen face image and the closest match will be returned.

4. Users can be added again by calling `/add_user` and deleted by calling `/delete_user` with the respective `uid`.   
**Make sure to call `/init` again after making any changes.**


## Add User

#### Endpoint:
**POST** `/add_user`

#### Query Parameters :
parameter|importance |Options|default| description |
---|---|---|---|---|
det |optional|0 or 1|1|whether to perform face detection or not. If `0` server expects you to provide a croped face|

> **Note:**  
> Query params are appended in the url  
> Example : `http://localhost:5000/add_user?det=1`

#### Body Parameters:
parameter| importance | field | description |
--- | --- |---|----|
uid | required |text |The unique identifier for an user identity|    
img | required | binary file |image file containing the face of the user|
name| required | text |name given to the image file|
option1|optional| text| optional value associated to an user|
option2|optional| text| optional value associated to an user|

### Repsonse:
```
{
    "success": true,
    "message": "Image added"
}
```


## Delete User
#### Endpoint:

**POST** /delete_user

#### Parameters:
parameter| importance | field | description |
--- | --- |---|----|
uid | required |text |The unique identifier for an user identity|

#### Response:
```
{
    "success": true,
    "message": "Successfully deleted"
}
```


## Build Index

#### Endpoint:
**GET** `/init`

#### Parameters:
*None*

#### Response:
```
Indexing Complete.
```
## Search User

#### Endpoint:
**POST** `/search_user`

#### Query Parameters :
parameter|importance |Options|default| description |
---|---|---|---|---|
det |optional|0 or 1|1|whether to perform face detection or not. If `0` server expects you to provide a croped face|

> **Note:**  
> Query params are appended in the url  
> Example : `http://localhost:5000/search_user?det=1`

#### Parameters:
parameter| importance | field | description |
--- | --- |---|----|
det | optional|int| 0 or 1 whether to perform face detection or not|
img | required | binary file |image file with a face to search for |

#### Response:

```
{
    "success": true,
    "data": {
        "uid": "03bcc1a9cf1b2be5371eb18f790d464b",
        "name": "anthony_hopkins",
        "distance": 0.78127575
    }
}
```

## Error and Warnings 

### Server Side :

Error Message | Description |
---|---|
Error verifying license|Connection issue - Please check your internet connection|
License expired|Allowed API call credits exhausted

### API Error Responses: 
API Error messages are returned with `success` field `False` and the `message` field carries the error message.


#### Add user

Error Message | Code |Description |
---|---|---|
Please provide a valid unique identifier|400|uid field is empty|
Error Fetching Image| 400| Input format is incorrect
Multiple faces in in image|400| Images should only contain a single face
No face detected in image|400| Either the image contains no face or a face could not be detected
Opening db failed| 500|Database access failed|
Adding image failed|500|Database read wrie issue|

#### Search_user

Error Message | Code |Description |
---|---|---|
Db row error|500|Could not retrive user info from db|
Query scan error|500|Could not retrive user info from db|
Marshall error|500|JSON matshalling failed|


#### Delete_user

Error Message | Code |Description |
---|---|---|
No user found|404|Please verify if uid is correct
Db query error|500|
Db query execution error|500|
Db query deletion error|500|

