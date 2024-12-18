# **HousingAnywhere Feed Specifications**
## **Using our feed**
This document outlines the specifications for supplying listings to Housing Anywhere, so they can be published on the [housinganywhere.com](https://housinganywhere.com/) website. This format follows the JSON specification, so only valid JSON data can be accepted.
### **Usage example:**
1. A HousingAnywhere partner has several listings they want to publish on the HousingAnywhere website.
1. They create a feed in the specified format with all the listings they want to share. The partner must supply a public url where the feed is exposed in JSON format.
1. The given URL will be added to the HousingAnywhere importer service.
1. HousingAnywhere's importer service will pull the feed every day. It will then process it accordingly:
   1. **Create:** For each new unique listing for this partner (based on the “ListingReference”), a new listing will be published.
   1. **Update:** If a listing with the same “ListingReference” is found, the existing listing will be updated.
   1. **Delete:** If an existing listing is no longer found it will be de-listed from the HousingAnywhere website.
## **Feed Fields**

|     **Name**      |                       **Type**                       |                                                                                                                                              **Description**                                                                                                                                               |                              **Required**                              |
|:-----------------:|:----------------------------------------------------:|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------:|:----------------------------------------------------------------------:|
| listingReference  |             String (max 100 characters)              |                                                                                                                              A unique (for this partner) id for this listing.                                                                                                                              |                               ![(tick)]                                |
|       type        |                 String (enumeration)                 |                                                                                                                The type of a listing. Possible `values`: `house`, `building`, `apartment`.                                                                                                                 |                               ![(tick)]                                |
|       kind        |                 String (enumeration)                 |                                                                                                          The kind of a listing. Possible values: `entire_place`, `private_room`, `shared_room. `                                                                                                           |                               ![(tick)]                                |
|      address      |                        String                        |                                                 The full street address including country, city and the house number of the listing.</p><p>The desired format is: `street`, `housenumber`, `city`, `state`, `neighbourhood`, `postalCode`, `countryCode`.                                                  |                               ![(tick)]                                |
|    description    |                        String                        |                                                                                                         The full description of the listing in plain text. Should be at least 50 characters long.                                                                                                          |                               ![(tick)]                                |
|       alias       |                  String (optional)                   |                                                                                                                                               Listing alias                                                                                                                                                |                                                                        |
|    pricingType    |                 String (enumeration)                 |                                                                                                                    <p>Possible values: `flat`, `monthly`. Default value is `flat`.</p>                                                                                                                     |                               ![(tick)]                                |
|     flatPrice     |                       Integer                        |                                                                                                                                           Monthly rent in cents                                                                                                                                            |  ![(tick)] Required based on the pricingType (if its set to `flat`).   |
|   monthlyPrices   |             List [MonthPrice Structure]              |                                                                                                        <p>Variable monthly rent as a list of MonthPrice structure.</p><p>See an example below.</p>                                                                                                         | ![(tick)] Required based on the pricingType (if its set to `monthly`). |
|       costs       | Mapping [key: Mapping [ cost type : cost Structure]] |                                                                                                 A mapping of listing costs (all other costs except monthly rent). For specifications see the next chapter.                                                                                                 |          ![(tick)] (See details in the cost structure below)           |
|   currencyCode    |                        String                        |                                                                                                                                ISO 4217 currency code. For example: “EUR”.                                                                                                                                 |                               ![(tick)]                                |
|     startDate     |                 String (YYYY-MM-DD)                  |                                                                                                                                 Date when this listing will be available.                                                                                                                                  |                               ![(tick)]                                |
|      endDate      |            String (YYYY-MM-DD) (optional)            |                                                                                                          Date when this listing will no longer be available. Will show as “unlimited” if not set.                                                                                                          |                                                                        |
|  blockedPeriods   |            List [BlockedPeriodStructure]             |                                                   Preferred way to indicate blocked periods. Used If the listings can have more than one unavailable period. If present and has at least one BlockedPeriod, then `startDate` and `endDate` are ignored.                                                    |             See details in Availability structure bellow.              |
| minimumStayMonths |                       Integer                        |                                                                                                                                    The minimal rental period in months.                                                                                                                                    |                                                                        |
| maximumStayMonths |                       Integer                        |                                                                                                                                    The maximum rental period in months.                                                                                                                                    |                                                                        |
|   moveInWindow    |                       Integer                        |                Setting this field to greater than 0 creates a time window in which in the listing is bookable. If the first day available is `StartDate` then the listing is bookable between `StartDate` and `StartDate` + `moveInWindow`. Setting it to 0 denotes no limit. Default is 0.                |                                                                        |
|    facilities     |           Mapping [key: FacilityStructure]           |                                                                                    A mapping with all currently supported facilities and whether this listing has them or not. For specifications see the next chapter.                                                                                    |       ![(tick)] (See details in the Facilities Structure below)        |
|      images       |                    Array [string]                    | An array of one or more images of the listing. URLs should point directly to the JPEG image. Each image will be downloaded and scaled/compressed if needed. Our version of these images will be served to our website visitors. Minimum image size is 0,5 megapixels. Maximum file size per image is 50MB. |                               ![(tick)]                                |
|    lastUpdated    |                  String (timestamp)                  |                                                                                           A timestamp that is last updated on any change in the listing fields. Format is `YYYY-MM-DD hh:mm:ss` in UTC timezone.                                                                                           |                               ![(tick)]                                |
|    listingUrl     |                        string                        |                                                                                                                              The url of the property/listing from the partner                                                                                                                              |                                                                        |
|      minAge       |                       Integer                        |                                                                                                      If there is a minimum age applicable to the tenant, i.e., they have to be at least 18 years old.                                                                                                      |                                                                        |
|      maxAge       |                       Integer                        |                                                                                                     If there is a maximum age applicable to the tenant, i.e., they have to be no older than 35 years.                                                                                                      |                                                                        |
| payoutMethodName  |                        string                        |                                                                                Payout method name. See the payout methods section below for more info. This appears as `account holder's name` in your *“Payout Settings”*.                                                                                |                                                                        |
## **Availability**
We support two methods to indicate when a listing is available.

If the listing has a date when becomes available or just an interval the best option is to use startDate and optionally endDate.

If the listing has multiple periods when it is unavailable, it is best to use `blockedPeriods` field. The BlockedPeriod structure has a startDate and an endDate. The date format must be `YYYY-MM-DD`.

The preferred way is to use the `blockedPeriods` field.

E.g.
```
{
    "listingReference": "123456789",
    ...
        "blockedPeriods": [
        {
            "startDate": "2019-01-01",
            "endDate": "2019-01-31"
        },
        {
            "startDate": "2019-05-01",
            "endDate": "2019-07-28"
        }
    ],
    ...
}
```

__Note: If `blockedPeriods` field is present, startDate and endDate are ignored.__
## **Price**
We support two price variations, flat and monthly. The flat price remains unchanged throughout the listing lifecycle. Monthly prices are variable per month, and can be set as follows,

List [MonthPrice Structure]

MonthPrice is composed of “price”, in cents (int) and “month”, denoting the number of the month (string).

E.g.
```
[
    {
        `"month": "1",
        `"Price": 80000
    },
    {
    	`"month": "2",
    	`"price": 80000
    },
    {
    	`"month": "3",
    	`"price": 90000
    },
    {
    	`"month": "4",
    	`"price": 90000
    },
    {
    	`"month": "5",
    	`"price": 90000
    },
    {
    	`"month": "6",
    	`"price": 80000
    },
    {
    	`"month": "7",
    	`"price": 70000
    },
    {
    	`"month": "8",
    	`"price": 60000
    },
    {
    	`"month": "9",
    	`"price": 60000
    },
    {
    	`"month": "10",
    	`"price": 60000
    },
    {
    	`"month": "11",
    	`"price": 50000
    },
    {
    	`"month": "12",
    	`"price": 50000
    }
]    
```

**Note - If the monthly price structure is chosen, make sure to provide values for all 12 months.**
## **Costs**
All other costs, except monthly rent, are presented as a mapping of cost objects.

The cost structure is to be built like
```
{
    // required cost types
    "costType": {
    	`"value": Integer,
    	`"paybleBy": String,
    	`"payableAt": String,
    	`"refundable": Boolean,
    	`"isEstimated": Boolean,
    	`"required": Boolean
    },
    "costType": {
    	`"value": Integer,
    	`"paybleBy": String,
    	`"payableAt": String,
    	`"refundable": Boolean,
    	`"isEstimated": Boolean,
    	`"required": Boolean
    }
    ...
    ...
    // optional costs if any
}
```
Please note, some **cost types** are required i.e. either the tenant is enforced to pay these or they are included in the rent. Even if the cost is included in the rent, the cost object must be declared stating `payableBy` = `included_in_rent`. The default value for required cost types is `not-applicable`. This means that for the cases when it is not set in the structure, the cost is omitted.
### **Required Costs**

|   **Key**   |       **Type**       |                                                                                                               **Description**                                                                                                               | **Required** |
|:-----------:|:--------------------:|:-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------:|:------------:|
|    type     | String (enumeration) | <p>The type of costs.</p><p>`administration-fee`</p><p>`electricity-bill`</p><p>`gas-bill`</p><p>`internet-bill`</p><p>`water-bill`</p><p>`utility-bills`</p><p>`cleaning-fee`</p><p>`broadcasting-bill`</p><p>`other-additional-costs`</p> |  ![(tick)]   |
|    value    |       Integer        |                                                                                     Value in cents, should be 0 when `PayableBy` is not set to tenant.                                                                                      |  ![(tick)]   |
|  payableBy  | String (enumeration) |                                                         <p>Possible values:</p><p>`tenant`,</p><p>`included_in_rent`,</p><p>`not_applicable`.</p><p>Default value is `tenant`.</p>                                                          |  ![(tick)]   |
|  payableAt  | String (enumeration) |                                      <p>Possible values:</p><p>`move-in`,</p><p>`move-out`,</p><p>`one-off`,</p><p>`monthly`,</p><p>`bi-weekly`,</p><p>`weekly`.</p><p>Default value is `monthly`.</p>                                      |  ![(tick)]   |
| refundable  |       Boolean        |                                                                                                      Possible values: `true`, `false`.                                                                                                      |  ![(tick)]   |
| isEstimated |       Boolean        |                                                                                                      Possible values: `true`, `false`.                                                                                                      |  ![(tick)]   |
### **Optional Costs**
Costs which can be made optional for the tenant depending on the service they use.

Example Use Case: There could be a towel deposit that the advertiser can choose to collect or not. This is marked by the `Required` field in the Cost Object.

|   **Key**   |       **Type**       |                                                                                                                                                                                                                                                         **Description**                                                                                                                                                                                                                                                          | **Required** |
|:-----------:|:--------------------:|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------:|:------------:|
|    type     | String (enumeration) | <p>The type of a cost. Possible values:</p><p>Deposits:</p><p>`bedding-deposit`</p><p>`other-deposit`</p><p>`security-deposit`</p><p>`towel-deposit`</p><p>Others:</p><p>`early-move-in`</p><p>`early-move-out`</p><p>`final-cleaning`</p><p>`late-move-in`</p><p>`late-move-out`</p><p>`other-optional`</p><p>`overnight-guests`</p><p>`cleaning-service`</p><p>`gym`</p><p>`towels-and-bedding`</p><p>`full-board`</p><p>`half-board`</p><p>`parking`</p><p>`bike-rent`</p><p>`end-early-fee`</p><p>`other-optional-costs`</p> |  ![(tick)]   |
|    value    |       Integer        |                                                                                                                                                                                                                                Value in cents, should be 0 when `PayableBy` is not set to tenant.                                                                                                                                                                                                                                |  ![(tick)]   |
|  payableBy  | String (enumeration) |                                                                                                                                                                                                    <p>Possible values:</p><p>`tenant`,</p><p>`included_in_rent`,</p><p>`not_applicable`.</p><p>Default value is `tenant`.</p>                                                                                                                                                                                                    |  ![(tick)]   |
|  payableAt  | String (enumeration) |                                                                                                                                                                                <p>Possible values:</p><p>`move-in`,</p><p>`move-out`,</p><p>`one-off`,</p><p>`monthly`,</p><p>`bi-weekly`,</p><p>`weekly`.</p><p>Default value is `monthly`.</p>                                                                                                                                                                                 |  ![(tick)]   |
|  required   |       Boolean        |                                                                                                                                                                                                                                                Possible values: `true`, `false`.                                                                                                                                                                                                                                                 |  ![(tick)]   |
| refundable  |       Boolean        |                                                                                                                                                                                                                                                Possible values: `true`, `false`.                                                                                                                                                                                                                                                 |  ![(tick)]   |
| isEstimated |       Boolean        |                                                                                                                                                                                                                                                Possible values: `true`, `false`.                                                                                                                                                                                                                                                 |  ![(tick)]   |
### **Example**
See the feed example below for the Costs element.

\*The required fields are enforced only when the cost structure is instantiated. If the partner chooses not to present the cost, the entire cost structure can be omitted.
## **Facilities**
These are all the facilities and the possible values for each facility currently supported.

- If a facility is not listed in the feed it will be set to the default.
- The default for each facility is null which means “Unknown” (will not be shown on the User Interface).
- Even though most of these fields are not required, it is critical to set them since it significantly improves the listing completeness score, and leads to higher conversions and more bookings.

|       **Key**        |       **Type**       |                                    **Description**                                     | **Required** |
|:--------------------:|:--------------------:|:--------------------------------------------------------------------------------------:|:------------:|
|   allergyFriendly    |       Boolean        |                        Possible values: `null`, `true`, `false`                        |              |
|   housematesGender   | String (enumeration) |      Possible Values: `null`, `shared_w_both`, `shared_w_boys`, `shared_w_girls`       |              |
| registrationPossible |       Boolean        |                        Possible values: `null`, `true`, `false`                        |              |
|     tenantStatus     | String (enumeration) | Possible Values: `null`, `working`, `working_student`, `student`, `looking_for_a_job`  |              |
|     bedroomCount     |       Integer        |                                                                                        |  ![(tick)]   |
|   bedroomFurnished   |       Boolean        |                        Possible values: `null`, `true`, `false`                        |  ![(tick)]   |
|     bedroomSize      |       Integer        |                                     Square meters                                      |  ![(tick)]   |
|     bedroomLock      |       Boolean        |                        Possible values: `null`, `true`, `false`                        |              |
|    balconyTerrace    | String (enumeration) |                   Possible Values: `null`, `no`, `shared`, `private`                   |              |
|       basement       | String (enumeration) |                   Possible Values: `null`, `no`, `shared`, `private`                   |              |
|       bathroom       | String (enumeration) | Possible Values: `null`, `shared_w_both`, `shared_w_boys`, `shared_w_girls`, `private` |              |
|        garden        | String (enumeration) |                   Possible Values: `null`, `no`, `shared`, `private`                   |              |
|       kitchen        | String (enumeration) |                   Possible Values: `null`, `no`, `shared`, `private`                   |              |
|      livingRoom      | String (enumeration) |                   Possible Values: `null`, `no`, `shared`, `private`                   |              |
|       parking        | String (enumeration) |                   Possible Values: `null`, `no`, `shared`, `private`                   |              |
|        toilet        | String (enumeration) |                   Possible Values: `null`, `no`, `shared`, `private`                   |              |
|     kitchenware      | String (enumeration) |                   Possible Values: `null`, `no`, `shared`, `private`                   |              |
|      totalSize       |       Integer        |                                     Square meters                                      |  ![(tick)]   |
| wheelchairAccessible |       Boolean        |                        Possible values: `null`, `true`, `false`                        |              |
|          ac          |       Boolean        |                        Possible values: `null`, `true`, `false`                        |              |
|         bed          |       Boolean        |                        Possible values: `null`, `true`, `false`                        |              |
|        closet        |       Boolean        |                        Possible values: `null`, `true`, `false`                        |              |
|         desk         |       Boolean        |                        Possible values: `null`, `true`, `false`                        |              |
|      dishwasher      |       Boolean        |                        Possible values: `null`, `true`, `false`                        |              |
|        dryer         |       Boolean        |                        Possible values: `null`, `true`, `false`                        |              |
|       flooring       | String (enumeration) |   Possible Values: `null`, `other`, `laminate`, `carpet`, `stone`, `wood`, `plastic`   |              |
|       internet       |       Boolean        |                        Possible values: `null`, `true`, `false`                        |              |
| livingRoomFurnished  |       Boolean        |                        Possible values: `null`, `true`, `false`                        |              |
|          tv          |       Boolean        |                        Possible values: `null`, `true`, `false`                        |              |
|    washingMachine    |       Boolean        |                        Possible values: `null`, `true`, `false`                        |              |
|         wifi         |       Boolean        |                        Possible values: `null`, `true`, `false`                        |  ![(tick)]   |
|    animalAllowed     | String (enumeration) |                  Possible Values: `null`, `yes`, `no`, `discussable`                   |              |
|    cleaningCommon    |       Boolean        |                        Possible values: `null`, `true`, `false`                        |              |
|   cleaningPrivate    |       Boolean        |                        Possible values: `null`, `true`, `false`                        |              |
|      playMusic       | String (enumeration) |                  Possible Values: `null`, `yes`, `no`, `discussable`                   |              |
|    smokingAllowed    | String (enumeration) |           Possible Values: `null`, `yes`, `no`, `discussable`, `outsideOnly`           |              |
|   preferredGender    | String (enumeration) |                       Possible Values: `null`, `male`, `female`                        |              |
|    couplesAllowed    |       Boolean        |                        Possible values: `null`, `true`, `false`                        |              |
|   currentOccupancy   |       Integer        |                                                                                        |              |
|      freePlaces      |       Integer        |                                                                                        |              |
|       heating        | String (enumeration) |       Possible Values: `null`, `central`, `gas`, `woodStove`, `electrical`, `no`       |  ![(tick)]   |
## **Payout methods**
The payout methods information is optional. If no listing has any payout information, the payout information on HousingAnywhere will not be changed. If at least one listing has payout information, the import process will synchronize HousingAnywhre payout information with the information from the feed.

Any payout method present in the feed will be updated on HousingAnywher to reflect the feed information.

If only some listings have payout information, those will be set up with respective payout methods. For the listings without payout information, the import will remove any explicit association on HousingAnywhere, so those listings will be associated with the default payout method.
## **Feed Example**
This is an example feed result with 1 room in Copenhagen.
```
[{
    "type": "house",
    "kind": "entire_place",
    "address": "Klerkegade 25H, 1308 Copenhagen Municipality, København K, DK",
    "pricingType": "monthly",
    "monthlyPricing":{
    	{
            "month": "1",
            "price": 80000
    	},
    	{
    		"month": "2",
    		"price": 80000
    	},
    	{
    		"month": "3",
    		"price": 90000
    	},
    	{
    		"month": "4",
    		"price": 90000
    	},
    	{
    		"month": "5",
    		"price": 90000
    	},
    	{
    		"month": "6",
    		"price": 80000
    	},
    	{
    		"month": "7",
    		"price": 70000
    	},
    	{
    		"month": "8",
    		"price": 60000
    	},
    	{
    		"month": "9",
    		"price": 60000
    	},
    	{
    		"month": "10",
    		"price": 60000
    	},
    	{
    		"month": "11",
    		"price": 50000
    	},
    	{
    		"month": "12",
    		"price": 50000
    	}
    },
    "blockedPeriods": [
        {
            "startDate": "2019-01-01",
            "endDate": "2019-01-31"
        },
        {
            "startDate": "2019-05-01",
            "endDate": "2019-07-28"
        }
    ],	
    "minimumStayMonths": 3,
    "maximumStayMonths": 12, 
    "moveInWindow": 20,
    "currencyCode": "DKK",
    "alias": "the-corner-room",
    "description": "This is an example room description, should be 50 characters atleast.",
    "facilities": {
    	"ac": false,
    	"registrationPossible": true,
    	"tenantStatus": "student",
    	"bedroomCount": 3,
    	"tv": false,
    	"bed": false,
    	"desk": true,
    	"bedroomLock": true,
    	"wifi": true,
    	"dryer": false,
    	"closet": false,
    	"garden": "shared",
    	"toilet": "shared",
    	"balconyTerrace": "private",
    	"heating": "central",
    	"kitchen": "shared",
    	"parking": "shared",
    	"basement": "shared_w_both",
    	"bathroom": "shared_w_both",
    	"flooring": "wood",
    	"internet": true,
    	"dishwasher": true,
    	"playMusic": "yes",
    	"kitchenware": "shared",
    	"livingRoom": "shared",
    	"animalAllowed": "discussable",
    	"cleaningCommon": true,
    	"livingRoomFurnished": true,
    	"bedroomFurnished": true,
    	"smokingAllowed": "outsideOnly",
    	"washingMachine": true,
    	"allergyFriendly": false,
    	"cleaningPrivate": false,
    	"wheelchairAccessible": true,
    	"bedroomSize": 25,
    	"totalSize": 70
    },
    "images": [
    	"https://cdn.example.com/photos/1.jpg",
    	"https://cdn.example.com/photos/2.jpg",
    	"https://cdn.example.com/photos/3.jpg"
    ],
    "lastUpdated": "2015-02-03 18:08:23",
    "listingReference": "ABCD1234XYZ",
    "listingUrl": "https://example.com/rooms/roomid",
    "costs": {
    	"security-deposit": {
    		"value": 80000,
    		"payableBy": "tenant",
    		"payableAt": "move-in",
    		"refundable": true,
    		"isEstimated": false
    	},
    	"administration-fee": {
    		"value": 65000,
    		"payableBy": "tenant",
    		"payableAt": "move-in",
    		"refundable": false,
    		"isEstimated": false
    	},
    	"electricity-bill": {
    		"value": 80000,
    		"payableBy": "tenant",
    		"payableAt": "monthly",
    		"refundable": false,
    		"isEstimated": false
    	},
    	"gas-bill": {
    		"value": 50000,
    		"payableBy": "tenant",
    		"payableAt": "bi-weekly",
    		"refundable": false,
    		"isEstimated": false
    	},
    	"internet-bill": {
    		"value": 100000,
    		"payableBy": "tenant",
    		"payableAt": "monthly",
    		"refundable": false,
    		"isEstimated": false
    	},
    	"water-bill": {
    		"value": 80000,
    		"payableBy": "tenant",
    		"payableAt": "monthly",
    		"refundable": false,
    		"isEstimated": false
    	},
    	"other-additional-costs": {
    		"payableBy": "not-applicable"
    	},
    	"towel-deposit": {
    		"value": 8000,
		"payableBy": "tenant",
		"payableAt": "move-in",
		"refundable": true,
		"required": false,
		"isEstimated": false
	    }
    },
    "minAge": 23,
    "maxAge": 45,
    "payoutMethodName": "My Company Payout",
}]
```

[(tick)]: Aspose.Words.95836984-d6ce-411a-b053-806c9297ae77.001.png
