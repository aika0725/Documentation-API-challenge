# Lightning Strike API Challenge

This project was developed as a code challenge for StormGEO.

## Table of contents
- [Lightning Strike API Challenge](#lightning-strike-api-challenge)
  - [Table of contents](#table-of-contents)
  - [Prerequisites](#prerequisites)
  - [Relevant resources](#relevant-resources)
  - [Preparations](#preparations)
  - [Solution overview](#solution-overview)
    - [API Key authentication](#api-key-authentication)
    - [Implementation](#implementation)
    - [Usage](#usage)
    - [**Error Handling**](#error-handling)
  - [Testing endpoints](#testing-endpoints)
  - [Endpoints](#endpoints)
    - [GetStrikes](#getstrikes)
    - [GetStrikesByArea](#getstrikesbyarea)
    - [GetAreas](#getareas)
    - [GetArea](#getarea)
    - [GetCustomers](#getcustomers)
    - [GetCustomersById](#getcustomersbyid)
    - [CreateCustomer](#createcustomer)
    - [UpdateCustomer](#updatecustomer)
    - [DeleteCustomer](#deletecustomer)
  - [Future possible improvements](#future-possible-improvements)
  - [How to reach me](#how-to-reach-me)

## Prerequisites

-   Docker Desktop or some other way to run a MySQL database locally - other options are to install MySQL, to use a package like XAMPP, or your own hosting
-   MySQL Workbench or similar to work with the data
-   Visual Studio 2022 and .NET 6

## Relevant resources

-   Skeleton project with one working API method
-   Database structure in `example-data/db-structure.sql`
-   Example lightning strike data in `example-data/strikes.zip` (extract to `strikes.sql`)
-   Example areas data in `example-data/areas.sql`
-   **Customers data in in `example-data/customers.sql`**
-   You can find all relevant test data in the folder `example-data`.

## Preparations

-   Get MySQL running (run `docker-compose up` in the project directory)
-   Create the database structure (run queries in `db-structure.sql`)
-   Import `areas.sql`
-   Import `customer.sql` (MySQL Workbench - Administration - Data import/Restore - Import from Self-Contained file)
-   Import `strikes.sql` (large file - use command line or MySQL Workbench Administration - Management - Data Import/Restore - Import from Self-Contained file)

## Solution overview

### API Key authentication
To ensure that only authorized customers can access the API endpoints, an API key authentication filter has been implemented for endpoints. This filter checks if the provided API key is valid and if the customer has access to the requested area.

### Implementation
The filter is implemented as a custom filter in the **`ApiKeyAuthFilter`** class.

The authentication process is as follows:

1.  Extract the API key from the request headers.
2.  Validate the API key by checking it against the customers in the database.
3.  If the API key is valid, verify that the customer has access to the requested area.
4.  If the API key is invalid or the customer does not have access to the area, return an appropriate error response.

### Usage

To use the API key authentication filter, add the `[ServiceFilter(typeof(ApiKeyAuthFilter))]` attribute to any controller or action that requires API key authentication.

For example:

```csharp
[HttpGet]
[ServiceFilter(typeof(ApiKeyAuthFilter))]
    public Strike[] GetStrikes(DateTime? startTime = null, DateTime? endTime = null, int? typeFilter = null)
    {
        var accessibleAreas = HttpContext.Items["AccessibleAreas"] as List<int>;

        return _lightningDb.GetStrikes(accessibleAreas ?? new List<int>(), startTime, endTime, typeFilter);
    }

```

CRUD for customers do not require API key authentication.

### **Error Handling**

The API key authentication filter handles the following error scenarios:

-   If the API key is not provided in the request headers, the filter returns a 401 Unauthorized status code with a custom error message.
-   If the API key is invalid, the filter returns a 403 Forbidden status code with a custom error message.

Other error handling:

-   If the customer does not have access to the requested area, it returns a 403 Forbidden status code with a custom error message.
-   If an item is not found, it returns a 404 Not Found status code.

## Testing endpoints

After everything is set up you can run the project and use swagger to test the API endpoints.

I recommend testing the API using **Swagger**, but if you want to test using Postman or some other tool, please keep in mind you have to include the apiKey as the following header: `X-Api-Key` .

Here’s a valid apiKey for testing: `KegGPD89VdUFr4a7aNFTL7nh` ; The allowed areas are: 2, 3.

## Endpoints
### GetStrikes
- **Endpoint**
	- `GET /api/LxArchive/GetStrikes`
-   **Description**
	- This endpoint retrieves lightning strikes based on the provided filters.
-   Parameters:
	- **`startTime`** (query, string): Start time for the range of strikes to retrieve, in the format **`yyyy-MM-ddTHH:mm:ss`**.
	-  **`endTime`** (query, string): End time for the range of strikes to retrieve, in the format **`yyyy-MM-ddTHH:mm:ss`**.
	- **`typeFilter`** (query, integer): Filter the strikes by their type. Accepted values are **`1`**, **`4, null.`** To include all strike types, don’t include the query.
-   **Example request**:
	- **GET**: `/api/LxArchive/GetStrikes?startTime=2022-09-28T19:50:06&endTime=2022-09-28T19:53:06&typeFilter=1`

### GetStrikesByArea
- **Endpoint**
	- `GET /api/LxArchive/GetStrikesByArea`
- **Description**
	- This endpoint retrieves lightning strikes based on the area ID and provided filters.
-   **Parameters**:
	- **`areaId`** (query, integer): areaId
	- **`startTime`** (query, string): Start time for the range of strikes to retrieve, in the format **`yyyy-MM-ddTHH:mm:ss`**.
	- **`endTime`** (query, string): End time for the range of strikes to retrieve, in the format **`yyyy-MM-ddTHH:mm:ss`**.
	- **`typeFilter`** (query, integer): Filter the strikes by their type. Accepted values are **`1`**, **`4, null.`** To include all strike types, don’t include the query.
-   **Example request**:
	- **GET**: `https://localhost:32776/api/LxArchive/GetStrikesByArea?areaId=4&typeFilter=1`

### GetAreas
- **Endpoint**
	- `GET /api/LxArchive/GetAreas`
-   **Description**
	- This endpoint retrieves all areas.
-   **Example request**:
	- **GET**: `https://localhost:32776/api/LxArchive/GetAreas`

### GetArea
- **Endpoint**
	- `GET /api/LxArchive/GetArea`
-   **Description**
	- This endpoint retrieves area based on the provided Area ID.

### GetCustomers
- **Endpoint**
	- `GET /api/LxArchive/GetCustomers`
-   **Description**
	- This endpoint retrieves all customers information

### GetCustomersById
- **Endpoint**
	- `GET /api/LxArchive/GetCustomersById/{id}`
-   **Description**
	- This endpoint retrieves a customers information based on the provided customer ID.
-   **Parameters**:
	- **`id`** (integer): Customer ID
-   **Example request**:
	- **GET**: `https://localhost:32774/api/LxArchive/GetCustomerById/503`

### CreateCustomer
- **Endpoint**
	- `GET /api/LxArchive/CreateCustomer`
-   **Description**
	- This endpoint creates a new customer.
-   **Example request**:
	- **POST**: `https://localhost:32776/api/LxArchive/CreateCustomer`
	-   **Request Body**: (customer ID and ApiKey will be generated automatically when a customer is created. Customer name is required and you can assign customer allowed areas when creating).
            `{ "name": "Customer Name", "allowedAreas": [1] }`

### UpdateCustomer
- **Endpoint**
	- `GET /api/LxArchive/UpdateCustomer/{id}`
-   **Description**
	- This endpoint updates the customer data
-   **Parameters**:
	- **`id`** (integer): Customer ID
-   **Example request**:
	- **PUT**: `https://localhost:32774/api/LxArchive/UpdateCustomer/503`
	-   **Request body**:
		-   _Update a customer:_
            `{ "name": "John", "apiKey": "aerawg$#1235", "allowedAreas": [2,3] }`
		-   _Update the customer-area relationships: (Update the allowedAreas from [2,3] to [3] )_
            `{ "allowedAreas": [3] }`

### DeleteCustomer
- **Endpoint**
	- `GET /api/LxArchive/DeleteCustomer/{id}`
-   **Description**
	-   This endpoint deletes a customer based on the provided customer Id.
-   **Parameters**:
	- **`id`** (integer): Customer ID
-   **Example request**:
	-   **DELETE**: **`https://localhost:32774/api/LxArchive/DeleteCustomer/503`**

## Future possible improvements

-   Add authentication for the customer endpoints (GetCustomers, CreateCustomer, etc…).
-   Create unit tests
-   Improve strikes table structure.

## How to reach me

If you’ve got any questions, feel free to contact me through my email: [kaiying0310@gmail.com](mailto:kaiying0310@gmail.com)
