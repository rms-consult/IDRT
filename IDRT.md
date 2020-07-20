# Interface for Demand Responsive Transport
This document specifies the endpoints and data that comprises the Interface for Demand Responsive Transport (IDRT).

## Term Definitions
This section defines terms that are used throughout this document.

* JSON - (JavaScript Object Notation) is a lightweight format for storing and transporting data. This document uses many terms defined by the JSON standard, including field, array, and object. (https://www.w3schools.com/js/js_json_datatypes.asp)
* Field - In JSON, a name/value pair consists of a field name (in double quotes), followed by a colon, followed by a value. (https://www.w3schools.com/js/js_json_syntax.asp)
* GeoJSON - GeoJSON is a format for encoding a variety of geographic data structures (https://geojson.org/). All geographic data use WGS84.
* Date - Date according to ISO 8601 (http://en.wikipedia.org/wiki/ISO_8601) "2020-01-23T18:25:43.511Z"
* Boolean - one of: "true", "false"
* Required - The field must be included in the dataset, and a value must be provided in that field for each record.
* Optional - The field may be omitted. An omitted field is equivalent to a field that is empty. If a parent element is optional the element itself and all it's child elements may be omitted regardless if the child elements are defined as required.
* Conditionally required - The field is required under certain conditions, which are outlined in the field description. Outside of these conditions, this field is optional.

## Booking Process
There are two distinct processes how DRT-Systems can handle the booking process:

### Explicit Booking
The booking is done explicitly by using the booking (POST) endpoint and might need to be confirmed using the confirmation (POST) endpoint. Neither are rides booked nor reserved in any form when using the availabilty endpoint.

### Implicit Booking
The booking is done implicitly by checking the availability for the requested ride. If this ride is available it is reserved automatically for a system specific time range and needs to be confirmed only. No seperated booking (POST) request is necessary.

## Authentication
For authentication an API key and secret are required across all endpoints. They must be present in the header of the requests as X-Api-Key and X-Api-Secret respectively. The API key is used for identifying the calling third-party-system too.

## Language
With the header parameter "Accept-Language" the preferred language of the third-party system for messages can be provided. This consists of a 2-3 letter base language tag representing the language, followed by sub-tags separated by ‘-‘ . The extra information is the region and country variant (like "en-US" or "fr-CA"). The default language is "en-US".

## Webhooks
Webhooks are used to inform the third-party system about specific events occurring in the DRT-System. When that event occurs, the DRT-system makes an HTTP request to the URL configured for the webhook.
The request should be acknowledged immediately with http status code 204.
It is assumed that the DRT-System has a management system for WebHooks that
* resends requests if no acknowledgment has been received
* eases the frequency of such repeated requests over time before finally dropping such requests
* drops oldest requests to prevent overloading the request backlog

### Messages
Webhook messages will be send by POST with the following header fields:
* The header field X-Webhook-Id will provide a unique id for every message that was sent. The id can be used to identify duplicate messages. If a message was not acknowledged it will be send again with the same id.
* Optionally when registering for a WebHook a signatureKey can be supplied. In this case the request will be hashed with the given signature key and added to the header field "X-Webhook-Signature". It can be used to verify the authentication of the request if no Api-Key is necessary for the MaaS-platform.

The message body contains the following information:

Field Name | Required | Type | Defines
--- | ---| --- | ---
`event` | Yes | String | Name of the event triggering the message.
`objectRef` | Yes | String | The reference of the object. Either a "bookingId" or a "availabilityId".
`object` | Yes | JSON-Object | The corresponding object to the event - always a booking object is returned.

### Events
Possible events are:

Event name | Required | Description 
--- | --- | ---
Booking:Status | Yes | The status of the booking has changed.
Booking:Change | Yes | Other parameters besides the status of the booking have changed, e.g. the assigned vehicle.
Availability:Monitor | Yes | The requested ride is available again.

## Endpoints DRT-system
The following endpoints are provided by the DRT-system and are used by the MaaS-Platform (RMVplus).

### Service (GET)
Defines the type, geographic area and service periode of available DRT-services.

Field Name | Required | Type | Defines
--- | ---| --- | ---
`point` | Optional | GeoJson POINT | Geographic point for which a DRT-service is searched.

#### Responses
http status code | Description 
--- | ---
200 | DRT-Service is available
400 | Invalid request
422 | No DRT-Service available
500 | Unexpected error

#### Response 200
Field Name | Required | Type | Defines
--- | ---| --- | ---
`serviceId` | Yes | String | Identifier of the DRT-service. This should be globally unique even between different systems.
`name` | Yes | String | Name of the DRT-service to be displayed e.g. to customers.
`shortName` | Yes | String | Short name of the DRT-service to be displayed e.g. to customers.
`description` | Optional | String | Additional information regarding the DRT-service which could be displayed e.g. to customers.
`operator` | Yes | String | Name of the operator to be displayed e.g. to customers.
`url` | Yes | String | The URL of the DRT-service providing more information for customers. The value must be a fully qualified URL that includes http:// or https://, and any special characters in the URL must be correctly escaped.
`logoUrl` | Yes | String | URL to the logo of the DRT-service to be displayed to the customer.
termsAndConditions | Yes | JSON-Object | Information regarding the terms and conditions.
`- releaseDate` | Yes | Date | Release date of the terms and conditions.
`- url` | Yes | String | URL where the terms and conditions can be accessed. The URL should point to a responsive HTML web site.
`preBookingMinutes` | Yes | Number | Number of minutes a booking can be placed into the future.
`bookingProcess` | Yes | String | Type of the booking process used. Either "explicit" or "implicit".
tariff | Yes | Array | Array of JSON-Objects describing the available tariffs.
`- id` | Yes | String | Identifier for the tariff.
`- name` | Yes | String | Name of the tariff to be displayed e.g. to customers.
`- description` | Yes | String | Description of the tariff to be displayed e.g. to customers.
services | Yes | Array | Array of JSON-Objects describing the spacial area, operating periods and type of the service.
`- maximumPassengersPerBooking` | Yes | Number | Maximum number of passengers for a single booking.
\- area | Conditionally required | GeoJSON FeatureCollection | A FeatureCollection (as described by the IETF RFC 7946). Spatially describing the area of service. Optionally if virtual stops are used.
`- - features` | Yes | Array | Array of FeatureObjects.
`- - - type` | Yes | String | "feature" (as per IETF RFC 7946).
`- - - geometry` | Yes | GeoJSON Multipolygon | A multipolygon that describes the service area. A clockwise arrangement of points defines the area enclosed by the polygon, while a counterclockwise order defines the area outside the polygon (right-hand rule).
\- virtualStops | Conditionally required | Array | Array of JSON-Objects. Trips can start and end at virtual stops. Optionally if service area is used.
`- - id` | Yes | String | Identifier of the virtual stop.
`- - name` | Yes | String | Name of the virtual stop.
`- - description` | Optional | String | Additional description of the virtual stop, e.g. "in front of 7-eleven".
`- - point` | Yes | GeoJSON Point | Geo-coordinates of the virtual stop.
\- operatingPeriods | Yes | Array | Array of JSON-Objects describing the time periods of the service operation.
`- - id` | Yes | String | Identifier of the operation period.
`- - start` | Yes | Date | Start date of the period.
`- - end` | Yes | Date | End date of the period.
\- bookableOptions | Optional | Array | Array of JSON-Objects describing the bookable options e.g. stroller.
`- - id` | Yes | String | Identifier of the option.
`- - name` | Yes | String | Name of the option e.g. stroller, wheelchair.
`- - maximumPerBooking` | Yes | String | Maximum number of options for a single booking.
customerApp | Yes | Array | Array of JSON-Objects providing customer app information.
`- os` | Yes | String | Identifier of the operating system, e.g. iOS.
`- storeUri` | Optional | String | A URI where the customer app can be downloaded from. Typically this will be a URI to an app store such as Google Play. If the URI points to an app store such as Google Play, the URI should follow Android best practices so an app can directly open the URI to the native app store app instead of a website. If a rentalUri field is populated then this field is required, otherwise it is optional.
`- discoveryUri` | Optional | String | A URI that can be used to discover if the customer app is installed on the device (e.g., using UIApplication canOpenURL:). This intent is used by apps prioritize customer apps for a particular user based on whether they already have a particular customer app installed. This field is required if a rentalUri field is populated, otherwise it is optional.

### Availability (GET)
Checks if a ride for the given origin, destination and time is possible. It is recommended to always use geo points for origin and destination even for services only operating between virtual stops. This allows the DRT-system to choose the most appropriate virtual stop even if this stop might not be the closest one. 

Field Name | Required | Type | Defines
--- | ---| --- | ---
`serviceID` | Yes | String | Identifier of the DRT-service.
`startPoint` | Conditionally required | GeoJson POINT | Start point of the trip. This field is required if startVirtualStop is not given.
`startVirtualStop` | Conditionally required | String | Virtual start stop of the trip. This field is required if startPoint is not given. If startPoint is given this field is ignored.
`endPoint` | Conditionally required | GeoJson POINT | End point of the trip. This field is required if endVirtualStop is not given.
`endVirtualStop` | Conditionally required | String | Virtual end stop of the trip. This field is required if endPoint is not given. If endPoint is given this field is ignored.
`departureTime` | Conditionally required | Date | Date and time when the trip should start. This field is required if arrivalTime is not given.
`arrivalTime` | Conditionally required | Date | Date and time when the trip should reach its destination. This field is required if departureTime is not given. If departureTime ist given this field is ignored.
`passengerNumber` | Yes | String | Number of passengers.
`maximumWalkingDistance` | Optional | String | Maximum walking distance in meter between start point of the trip and pick up location and between drop off location and end point.
`tarifID` | Yes | String | Identifier of the used tariff.
bookableOptions | Optional | Array | Array of JSON-Objects describing the bookable options e.g. stroller.
`- id` | Yes | String | Identifier of the option.
`- number` | Yes | String | Number of options to book.
`availabilityMonitor` | Optional | Boolean | If "yes" the availability will be monitored if the ride request can't  be fulfilled right now. If the requested ride can be served again a booking is created automatically and the given URL is called (Webhook). The created booking needs to be confirmed. In order to have this feature working the corresponding webhook "Availability:Monitor" must be subscribedn to.
`customerID` | Conditionally required | String | If the availablility is monitored an identifier of the customer is required. It will be used for the automatically created booking once the ride becomes available.

#### Responses
http status code | Description 
--- | ---
200 | Ride is available
400 | Invalid request
422 | Ride is not available
500 | Unexpected error

#### Response 200
Field Name | Required | Type | Defines
--- | ---| --- | ---
`startPoint` | Yes | GeoJson POINT | Start point of the trip.
`startAddress` | Optional | String | Address of the start point of the trip to be displayed to the customer.
`endPoint` | Yes | GeoJson POINT | End point of the trip.
`endAddress` | Optional | String | Address of the end point of the trip to be displayed to the customer.
departureTime | Yes | JSON-Object | JSON-Object providing the departure time.
`- earliest` | Yes | Date | Estimated earliest date and time of departure. It is strongly recommended to advice the customer to be present at the departure point at this time.
`- latest` | Yes | Date | Estimated latest date and time of departure.
arrivalTime | Yes | JSON-Object | JSON-Object providing the arrival time.
`- earliest` | Yes | Date | Estimated earliest date and time of arrival.
`- latest` | Yes | Date | Estimated latest date and time of arrival.
tripSegment | Yes | Array | Array of JSON-Objects providing information for each segment of the trip.
`- startPoint` | Yes | GeoJson POINT | Start point of the trip segment.
`- startAddress` | Optional | String | Address of the start point of the trip segment to be displayed to the customer.
\- departureTime | Yes | JSON-Object | JSON-Object providing the departure time.
`- - earliest` | Yes | Date | Estimated earliest date and time of departure.
`- - latest` | Yes | Date | Estimated latest date and time of departure.
`- endPoint` | Yes | GeoJson POINT | Start point of the trip segment.
`- endAddress` | Optional | String | Address of the end point of the trip segment to be displayed to the customer.
\- arrivalTime | Yes | JSON-Object | JSON-Object providing the arrival time.
`- - earliest` | Yes | Date | Estimated earliest date and time of arrival.
`- - latest` | Yes | Date | Estimated latest date and time of arrival.
`- mode` | Yes | String | Either "walk" or "ride".
`- distance` | Conditionally required | String | Distance of the segment in meters if it's a walking segment.
`- routeOutline` | Optional | GeoJson LineString | Visual representation of the trip segment as LineString to be displayed on a map to the customer. The outline is based on the map used by the DRT-system which might be different to the map used by the third-party system.
`price` | Yes | String | Price of the trip.
`currency` | Yes | String | Currency the price is in (ISO 4217 code: http://en.wikipedia.org/wiki/ISO_4217).
`bookingId` | Optional | String | Identification of the booking. Only if implicit booking is used.
bookingUrl | Yes | String | A JSON object that contains rental URLs (deep links).
`- os` | Yes | String | Identifier of the operating system.
`- uri` | Yes | String | This URI should be a deep link specific to this ride, and should not be a general page that includes general information for the DRT-service. 

#### Response 422
Field Name | Required | Type | Defines
--- | ---| --- | ---
`code` | Yes | String | Code, as listed below.
`description` | Yes | String | Human readable error description.
`availabilityId` | Optional | String | If an availability monitor was requested a reference is provided. When the ride will be available again this reference will be included in the webhook message. 

Code | Descritption 
--- | ---
100 | "Start or end point outside of service area."
101 | "Departure or arrival time outside of service operating times."
102 | "Ride request can't be satisfied at this time."
103 | "Start and end point of the ride are too close together."
104 | "No rides are possible between the requested start & end point and time."
105 | "Ride request can't be satisfied for this number of passengers."
106 | "The bookable option can not be satisfied: {name}."
200 | "Ride request can't be satisfied at this time. The availablility will be monitored."

### Customer (POST)
Transfers the customer data which is necessary to conclude a contract between DRT-service provider and customer. It is recommended to create shadow accounts only and keep customers in separate groups for each MaaS-platform.

Field Name | Required | Type | Defines
--- | ---| --- | ---
`customerID` | Yes | String | Identifier of the customer used by the MaaS-platform. Bookings are performed on behalf of the customer.
`firstName` | Yes | String | First name of the customer.
`lastName` | Yes | String | Last name of the customer.
Address | Optional | JSON-Object | JSON-Object with address information.
`- street` | Yes | String | Name of the street.
`- houseNumber` | Yes | String | House number.
`- city` | Yes | String | Name of the city.
`- postcode` | Yes | String | Postal code of the city.
`- country` | Yes | String | Name of the country.
`email` | Yes | String | Email-address of the customer.
`mobileNumber` | Optional | String | Mobile phone number.

#### Responses
http status code | Description 
--- | ---
204 | Transfer successful
400 | Invalid request
500 | Unexpected error

### Booking (POST)
Book a ride.

Field Name | Required | Type | Defines
--- | ---| --- | ---
`serviceID` | Yes | String | Identifier of the DRT-service.
`customerID` | Yes | String | Identifier of the customer on behalf the booking is performed.
`startPoint` | Conditionally required | GeoJson POINT | Start point of the trip. This field is required if startVirtualStop is not given.
`startVirtualStop` | Conditionally required | String | Virtual start stop of the trip. This field is required if startPoint is not given. If startPoint is given this field is ignored.
`endPoint` | Conditionally required | GeoJson POINT | End point of the trip. This field is required if endVirtualStop is not given.
`endVirtualStop` | Conditionally required | String | Virtual end stop of the trip. This field is required if endPoint is not given. If endPoint is given this field is ignored.
`departureTime` | Conditionally required | Date | Date and time when the trip should start. This field is required if arrivalTime is not given.
`arrivalTime` | Conditionally required | Date | Date and time when the trip should reach its destination. This field is required if departureTime is not given. If departureTime is given this field is ignored.
`passengerNumber` | Yes | String | Number of passengers.
`maximumWalkingDistance` | Yes | String | Maximum walking distance in meter between start point of the trip and pick up location and between drop off location and end point.
`tarifID` | Yes | String | Identifier of the used tariff.
bookableOptions | Optional | Array | Array of JSON-Objects describing the bookable options e.g. stroller.
`- id` | Yes | String | Identifier of the option.
`- number` | Yes | String | Number of options to book.
`type` | Yes | String | Either "booking" or "reservation". If the trip is reserved it is not booked yet. It is only reserved for a system specific time and it's not bookable for others. It must be confirmed within the specified time frame. If not confirmed the reservation will be released.

#### Responses
http status code | Description 
--- | ---
200 | Booking/Reservation successful
400 | Invalid request
422 | Booking/Reservation was not possible
500 | Unexpected error

#### Response 200
Field Name | Required | Type | Defines
--- | ---| --- | ---
`bookingId` | Yes | String | Identification of the booking.
`startPoint` | Yes | GeoJson POINT | Start point of the trip.
`startAddress` | Optional | String | Address of the start point of the trip to be displayed to the customer.
`endPoint` | Yes | GeoJson POINT | End point of the trip.
`endAddress` | Optional | String | Address of the end point of the trip to be displayed to the customer.
departureTime | Yes | JSON-Object | JSON-Object providing the departure time.
`- earliest` | Yes | Date | Estimated earliest date and time of departure. It is strongly recommended to advice the customer to be present at the departure point at this time.
`- latest` | Yes | Date | Estimated latest date and time of departure.
arrivalTime | Yes | JSON-Object | JSON-Object providing the arrival time.
`- earliest` | Yes | Date | Estimated earliest date and time of arrival.
`- latest` | Yes | Date | Estimated latest date and time of arrival.
`passengerNumber` | Yes | String | Number of passengers.
`bookingCode` | Optional | String | Code for this booking. The code e.g. can be used to identify the customer at pick up.
`maximumTimeWindow` | Yes | String | Difference between estimated departure time and latest departure time in minutes.
tripSegment | Yes | Array | Array of JSON-Objects providing information for each segment of the trip.
`- startPoint` | Yes | GeoJson POINT | Start point of the trip segment.
`- startAddress` | Optional | String | Address of the start point of the trip segment to be displayed to the customer.
\- departureTime | Yes | JSON-Object | JSON-Object providing the departure time.
`- - earliest` | Yes | Date | Estimated earliest date and time of departure.
`- - latest` | Yes | Date | Estimated latest date and time of departure.
`- endPoint` | Yes | GeoJson POINT | Start point of the trip segment.
`- endAddress` | Optional | String | Address of the end point of the trip segment to be displayed to the customer.
\- arrivalTime | Yes | JSON-Object | JSON-Object providing the arrival time.
`- - earliest` | Yes | Date | Estimated earliest date and time of arrival.
`- - latest` | Yes | Date | Estimated latest date and time of arrival.
`- mode` | Yes | String | Either "walk" or "ride".
`- distance` | Conditionally required | String | Distance of the segment in meters if it's a walking segment.
`- routeOutline` | Yes | GeoJson LineString | Visual representation of the trip segment as LineString to be displayed on a map to the customer. The outline is based on the map used by the DRT-system which might be different to the map used by the third-party system.
price | Yes | JSON-Object | JSON-Object with information regarding the price of the trip.
`- amount` | Yes | String | Price of the trip.
`- currency` | Yes | String | Currency the price is in (ISO 4217 code: http://en.wikipedia.org/wiki/ISO_4217).
vehicle | Yes | JSON-Object | JSON-Object with information to the scheduled vehicle and driver.
`- description` | Optional | String | Description of the scheduled vehicle e.g. brand and model.
`- number` | Optional | String | Number of the scheduled vehicle.
`- licensePlate` | Optional | String | License plate of the scheduled vehicle.
`- pictureUrl` | Optional | String | URL where a picture of the scheduled vehicle can be retrieved.
`- driverName` | Optional | String | Name of the driver.
`liveApproachUrl` | Conditional required | String | If the vehicle is on the way to the pick up location the provided URL gives live updates regarding the approach particularly the position of the vehicle and the ETA. The URL is intended to be used by apps in order to visualize the approach of the vehicle to the customer.

#### Response 422
Field Name | Required | Type | Defines
--- | ---| --- | ---
`code` | Yes | String | Code, as listed below.
`description` | Yes | String | Human readable error description.

Code | Descritption 
--- | ---
100 | "Start or end point outside of service area."
101 | "Departure or arrival time outside of service operating times."
102 | "Ride request can't be satisfied at this time."
103 | "Start and end point of the ride are too close together."
104 | "No rides are possible between the requested start & end point and time."
105 | "Ride request can't be satisfied for this number of passengers."
106 | "The bookable option can not be satisfied: {name}."

### Booking (DELETE)
Cancel a previously made booking or reservation.

Field Name | Required | Type | Defines
--- | ---| --- | ---
`bookingId` | Yes | String | Identification of the booking.

#### Responses
http status code | Description 
--- | ---
204 | Cancellation successful
400 | Invalid request
404 | Booking not found
500 | Unexpected error

### Booking (GET)
Query all information regarding the given booking.

Field Name | Required | Type | Defines
--- | ---| --- | ---
`bookingId` | Yes | String | Identification of the booking.

#### Responses
http status code | Description 
--- | ---
200 | Query successful
400 | Invalid request
500 | Unexpected error

#### Response 200
Field Name | Required | Type | Defines
--- | ---| --- | ---
`startPoint` | Yes | GeoJson POINT | Start point of the trip.
`startAddress` | Optional | String | Address of the start point of the trip to be displayed to the customer.
`endPoint` | Yes | GeoJson POINT | End point of the trip.
`endAddress` | Optional | String | Address of the end point of the trip to be displayed to the customer.
departureTime | Yes | JSON-Object | JSON-Object providing the departure time.
`- earliest` | Yes | Date | Estimated earliest date and time of departure. It is strongly recommended to adivce the customer to be present at the departure point at this time.
`- latest` | Yes | Date | Estimated latest date and time of departure.
arrivalTime | Yes | JSON-Object | JSON-Object providing the arrival time.
`- earliest` | Yes | Date | Estimated earliest date and time of arrival.
`- latest` | Yes | Date | Estimated latest date and time of arrival.
`passengerNumber` | Yes | String | Number of passengers.
`bookingCode` | Optional | String | Code for this booking. The code e.g. can be used to identify the customer at pick up.
`maximumTimeWindow` | Yes | String | Difference between estimated departure time and latest departure time in minutes.
tripSegment | Yes | Array | Array of JSON-Objects providing information for each segment of the trip.
`- startPoint` | Yes | GeoJson POINT | Start point of the trip segment.
`- startAddress` | Optional | String | Address of the start point of the trip segment to be displayed to the customer.
\- departureTime | Yes | JSON-Object | JSON-Object providing the departure time.
`- - earliest` | Yes | Date | Estimated earliest date and time of departure.
`- - latest` | Yes | Date | Estimated latest date and time of departure.
`- endPoint` | Yes | GeoJson POINT | Start point of the trip segment.
`- endAddress` | Optional | String | Address of the end point of the trip segment to be displayed to the customer.
\- arrivalTime | Yes | JSON-Object | JSON-Object providing the arrival time.
`- - earliest` | Yes | Date | Estimated earliest date and time of arrival.
`- - latest` | Yes | Date | Estimated latest date and time of arrival.
`- mode` | Yes | String | Either "walk" or "ride".
`- distance` | Conditionally required | String | Distance of the segment in meters if it's a walking segment.
`- routeOutline` | Yes | GeoJson LineString | Visual representation of the trip segment as LineString to be displayed on a map to the customer. The outline is based on the map used by the DRT-system which might be different to the map used by the third-party system.
price | Yes | JSON-Object | JSON-Object with information regarding the price of the trip.
`- amount` | Yes | String | Price of the trip.
`- currency` | Yes | String | Currency the price is in (ISO 4217 code: http://en.wikipedia.org/wiki/ISO_4217).
vehicle | Yes | JSON-Object | JSON-Object with information to the scheduled vehicle and driver.
`- description` | Optional | String | Description of the scheduled vehicle e.g. brand and model.
`- number` | Optional | String | Number of the scheduled vehicle.
`- licensePlate` | Optional | String | License plate of the scheduled vehicle.
`- pictureUri` | Optional | String | Picture of the scheduled vehicle.
`- driverName` | Optional | String | Name of the driver.
`status` | Yes | String | Status of the booking. At least the following status should be supported: "reserved", "booked", "scheduled", "in approach", "arrived", "on route", "done", "no show", "canceled by system", "canceled by user".

### Confirmation (POST)
Confirm a previously made reservation. The reservation is turned into a booking.

Field Name | Required | Type | Defines
--- | ---| --- | ---
`bookingId` | Yes | String | Identification of the booking.

#### Responses
http status code | Description 
--- | ---
204 | Confirmation successful
400 | Invalid request
500 | Unexpected error

### Subscription (POST)
Creat a webhook for the specified type of events.

Field Name | Required | Type | Defines
--- | ---| --- | ---
`event` | Yes | Array | String array of events.
`url` | Yes | String | URL to be called.
`signatureKey` | Optional | String | Signature that is used for hashing the request.

#### Response
http status code | Description 
--- | ---
200 | Subscription created
400 | Invalid request
500 | Unexpected error

#### Response 200
Field Name | Required | Type | Defines
--- | ---| --- | ---
`subscriptionId` | Yes | String | Identifier of the subscription.

### Subscription (DELETE)
Delete a webhook.

Field Name | Required | Type | Defines
--- | ---| --- | ---
`subscriptionId` | Yes | String | Identifier of the subscription.

#### Response
http status code | Description 
--- | ---
200 | Subscription deleted
400 | Invalid request
500 | Unexpected error

### Invoice (GET)
Fetch invoices which have been generated by the DRT-system.

Field Name | Required | Type | Defines
--- | ---| --- | ---
`dateStart` | Yes | String | Fetch invoices between start and end date.
`dateEnd` | Yes | String | Fetch invoices between start and end date.

#### Responses
http status code | Description 
--- | ---
200 | Successful
400 | Invalid request
500 | Unexpected error

#### Response 200
Field Name | Required | Type | Defines
--- | ---| --- | ---
`customerID` | Yes | String | Identifier of the customer used by the MaaS-platform.
`invoiceID` | Yes | String | Identifier of this invoice.
`date` | Yes | String | Date the invoice was created.
`totalPrice` | Yes | String | Total price of this invoice.
`currency` | Yes | String | Currency the totalPrice is in (ISO 4217 code: http://en.wikipedia.org/wiki/ISO_4217).
items | Yes | Array | Array of bookings included in this invoice.
`- bookingId` | No | String | Identification of the booking if the invoice item is related to a booking. A refund might not be related to a specific booking.
`invoiceUrl` | Yes | String | URL where the invoice can be accessed e.g. in pdf-format.

### PaymentStatus (POST)
Change the payment status of an invoice. 

Field Name | Required | Type | Defines
--- | ---| --- | ---
`customerID` | Yes | String | Identifier of the customer used by the MaaS-platform.
`invoiceID` | Yes | String | Identifier of this invoice.
`status` | Yes | String | One of "paid", "dunned", "failed".

#### Responses
http status code | Description 
--- | ---
200 | Successful
400 | Invalid request
500 | Unexpected error

#### Response 400
Field Name | Required | Type | Defines
--- | ---| --- | ---
`code` | Yes | String | Code, as listed below.
`description` | Yes | String | Human readable error description.

Code | Descritption 
--- | ---
100 | "Customer unknown."
101 | "Invoice doesn't exist."
103 | "Invalid status."
