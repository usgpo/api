# api

The **govinfo** api is intended to provide data users with a simple means to programmatically access **govinfo** content and metadata, which is stored in self-describing packages. This initial release provides functionality to retrieve lists of packages added or modified within a given time frame, summary metadata for packages, direct access to content and metadata formats, and equivalent granule information.

Interactive documentation using the OpenAPI/swagger specification is available at https://api.govinfo.gov/docs.

## Keys

This API requires the use of an API.data.gov key - [signup here](https://api.data.gov/signup/). If you already have one, go to the [/docs page](https://api.govinfo.gov/docs), click on Authorize, and enter your key. Then you can make all the requests normally.

You can send your API key in a few different ways. See api.data.gov for more information on [key usage](https://api.data.gov/docs/api-key/).

## Versioning

One of the API's design goals is to minimize breaking changes, but we are implementing versioning to allow users to specify the version required. We have not determined the number of versions that we plan to support at this time, but it will be based on usage and we will communicate with known users prior to deprecating any version.

If no version is specified in requests, the latest version of the api will be provided.

## Issues

See [Issues](https://github.com/usgpo/api/issues) to submit an issue or comment on future development priorities.

See the labels below for additional information:

- [Features](https://github.com/usgpo/api/labels/Features) - list of features that are currently under consideration by the govinfo team. Your feedback is welcome!
- [Upcoming](https://github.com/usgpo/api/labels/Upcoming) - list of features that are currently under development and will be available in our next release. May also be tagged with a release label (Month Year) e.g. "July 2018".

## About the Data

Data available in the API represents official publications from all three branches of the Federal Government as made available on [GPO's](https://www.gpo.gov) [**govinfo**](https://www.govinfo.gov). For more information about the data that's available from **govinfo**, see our [about](https://www.govinfo.gov/about) page and [learn what's available](https://www.govinfo.gov/help/whats-available).

## Quickstart

### Collections Service | [samples](/samples/collections/)

#### Base collections request | [sample](/samples/collections/collections.json) | [formatted](/samples/collections/collections-formatted.json)

https://api.govinfo.gov/collections?api_key=DEMO_KEY

This request will provide a json list of the collections available within our system, including a `collectionCode`, `collectionName`, `packageCount`, and `granuleCount` (as applicable). All json response are returned in a minified format.

### Collection update

The following request allows you to specify a collection and get a list of packageIds that have been added or modified within the specified time period. `collectionCode` and `startDate` are required, as are `offset` and `pageSize`. Optionally, you can include the `endDate`. Currently, we are not limiting the `pageSize`, but we may restrict this based on performance needs.

>**Note:** There is a 10000 item limit on collections responses. This means that if your update range is too broad, you may need to narrow it down using the `endDate` parameter. [Example](https://github.com/usgpo/api/issues/19#issuecomment-428292313)

##### startDate/endDate usage
`startDate` and `endDate` are parameters used to search against the `lastModified` value for the individual packages. This represents the time that this package was added or updated - equivalent to the value listed in the sitemaps. It is not the equivalent to Date Published, Date Issued, or Date Ingested in MODS.

#### Congressional Bills with startDate only (BILLS) | [sample](/samples/collections/BILLS-sample.json) | [formatted](/samples/collections/BILLS-sample-formatted.json)

https://api.govinfo.gov/collections/BILLS/2018-01-01T00:00:00Z/?offset=0&pageSize=100&api_key=DEMO_KEY

#### Congressional Bills with endDate | [sample](/samples/collections/BILLS-sample-endDate.json) | [formatted](/samples/collections/BILLS-sample-endDate-formatted.json)

https://api.govinfo.gov/collections/BILLS/2018-07-03T00:00:00Z/2018-07-10T23:59:59Z?offset=150&pageSize=150&api_key=DEMO_KEY

#### Congressional Record (CREC) | [sample](/samples/collections/CREC-sample.json) | [formatted](/samples/collections/CREC-sample-formatted.json)

https://api.govinfo.gov/collections/CREC/2018-07-01T00:00:00Z?offset=0&pageSize=10&api_key=DEMO_KEY

#### United States Court Opinions (USCOURTS) | [sample](/samples/collections/USCOURTS-sample.json) | [formatted](/samples/collections/USCOURTS-sample-formatted.json)

https://api.govinfo.gov/collections/USCOURTS/2018-04-03T00:00:00Z?offset=0&pageSize=25&api_key=DEMO_KEY

### Published Service

This is similar to the _collections_ service in that it provides users with an easy way to get a list of packages by date. The **difference** is that this service provides packages based on dateIssued -- this generally corresponds to the publication date of the content itself, rather than the govinfo system update time for a publication. 

#### Format:

https:// api.govinfo.gov/published/`dateIssuedStartDate`/`dateIssuedEndDate`?offset=`startingRecord`&pageSize=`number of records in call`&collection=`comma-separated list of values`&api_key=`your api.data.gov api key`

#### Examples:

BILLS issued between January and July 2019:
https://api.govinfo.gov/published/2019-01-01/2019-07-31?offset=0&pageSize=100&collection=BILLS&api_key=DEMO_KEY 

Federal Register and CFR packages in 2019:
https://api.govinfo.gov/published/2019-01-01/2019-12-31?offset=0&pageSize=100&collection=CFR,FR&modifiedSince=2020-01-01T00:00:00&api_key=DEMO_KEY  


#### Required parameters
- `dateIssuedStartDate`: the earliest package you are requesting by dateIssued – YYYY-MM-DD
- `offset`: starting record – usually 0. If pageSize=10, you could advance to the next page of results by applying `offset=10`.
- `pageSize`: number of records to return per request (e.g. 10)
- `collection`: comma-separated list of collections that you are requesting, e.g. https://api.govinfo.gov/published/2019-01-01/2019-12-31?offset=0&pageSize=100&collection=BILLS,BILLSTATUS&api_key=DEMO_KEY  - see [/collections](https://api.govinfo.gov/collections?api_key=DEMO_KEY) for a list of collections by code and human-readable name.

#### Optional parameters:
- `dateIssuedEndDate`: the latest package you are requesting by dateIssued – YYYY-MM-DD
- `docClass`:  Filter the results by overarching collection-specific categories. The values vary from collection to collection. For example, docClass in BILLS corresponds with Bill Type --e.g. s, hr, hres, sconres. CREC (the Congressional Record) has docClass by CREC section:  HOUSE, SENATE, DIGEST, and EXTENSIONS
- `congress`: congress number (e.g. “116”)
- `modifiedSince`: equivalent to the startDate parameter in the collections service which is based on lastModified– allows you to request only packages that have been modified since a given date/time – useful for tracking updates. Requires ISO 8601 format -- e.g. 2020-02-28T00:00:00Z

### Packages service | [samples](/samples/packages/)

This service allows you to specify a **govinfo** `packageId` and retrieve available forms of content and metadata. A `/summary` json response is available that includes links and basic metadata about the package - generally equivalent to the information available on the details page for that package.

From the summary, you can get access to all available content and metadata formats for a package. Here is a sample download section for the BILLS-115hr1625enr example below, including a link to the related Billstatus package:

```json
"download": {
"txtLink": "https://api.govinfo.gov/packages/BILLS-115hr1625enr/htm",
"xmlLink": "https://api.govinfo.gov/packages/BILLS-115hr1625enr/xml",
"pdfLink": "https://api.govinfo.gov/packages/BILLS-115hr1625enr/pdf",
"modsLink": "https://api.govinfo.gov/packages/BILLS-115hr1625enr/mods",
"premisLink": "https://api.govinfo.gov/packages/BILLS-115hr1625enr/premis",
"zipLink": "https://api.govinfo.gov/packages/BILLS-115hr1625enr/zip"
},
"related": {
"billStatusLink": "https://www.govinfo.gov/bulkdata/BILLSTATUS/115/hr/BILLSTATUS-115hr1625.xml"
},
```

#### Congressional Bills (BILLS) | [sample](/samples/packages/BILLS-115hr1625enr-summary.json) | [formatted](/samples/packages/BILLS-115hr1625enr-summary-formatted.json)

https://api.govinfo.gov/packages/BILLS-115hr1625enr/summary?api_key=DEMO_KEY

#### Congressional Record (CREC) | [sample](/samples/packages/CREC-2018-01-03-summary.json) | [formatted](/samples/packages/CREC-2018-01-03-summary-formatted.json)

https://api.govinfo.gov/packages/CREC-2018-01-03/summary?api_key=DEMO_KEY

#### Federal Register (FR) | [sample](/samples/packages/FR-2018-04-12-summary.json) | [formatted](/samples/packages/FR-2018-04-12-summary-formatted.json)

https://api.govinfo.gov/packages/FR-2018-04-12/summary?api_key=DEMO_KEY

### Granule lists

You can also get a list of available granules for a specified package by adding `/granules`, `offset` and `pageSize`

#### Congressional Hearings (CHRG) | [sample](/samples/packages/granules/CHRG-107shrg82483-granules.json) | [formatted](/samples/packages/granules/CHRG-107shrg82483-granules.json)

https://api.govinfo.gov/packages/CHRG-107shrg82483/granules?offset=0&pageSize=10&api_key=DEMO_KEY

#### Congressional Record (CREC) | [sample](/samples/packages/granules/CREC-2018-01-03-granules.json) | [formatted](/samples/packages/granules/CREC-2018-01-03-granules.json)

https://api.govinfo.gov/packages/CREC-2018-01-03/granules?offset=0&pageSize=100&api_key=DEMO_KEY

#### Federal Register (FR)  | [sample](/samples/packages/granules/FR-2018-04-12-granules.json) | [formatted](/samples/packages/granules/CREC-2018-01-03-granules.json)

https://api.govinfo.gov/packages/FR-2018-04-12/granules?offset=0&pageSize=100&api_key=DEMO_KEY

This provides a list of titles, granuleIds and links to the granule summary, where you can access all available content and metadata formats, including the zip.

### Granules

Similar to the package summary, you can retrieve a json summary for any granule, which will return basic metadata as well as links to all available content and metadata.

#### Congressional Record (CREC) | [sample](/samples/packages/granules/CREC-2018-03-01-pt1-PgD211-granule-summary.json) | [formatted](/samples/packages/granules/CREC-2018-03-01-pt1-PgD211-granule-summary-formatted.json)

https://api.govinfo.gov/packages/CREC-2018-07-10/granules/CREC-2018-07-10-pt1-PgD782/summary?api_key=DEMO_KEY

#### Federal Register (FR) | [sample](/samples/packages/granules/FR-2018-04-12_2018-07777-granule-summary.json) | [formatted](/samples/packages/granules/FR-2018-04-12_2018-07777-granule-summary-formatted.json)

https://api.govinfo.gov/packages/FR-2018-04-12/granules/2018-07777/summary?api_key=DEMO_KEY

A number of collections have specific additional collection-specific metadata values included in the API response, such as social media information in the CDIR collection. Here are some examples:

https://api.govinfo.gov/packages/CDIR-2018-10-29/granules/CDIR-2018-10-29-DE-S-2/summary?api_key=DEMO_KEY

https://api.govinfo.gov/packages/CDIR-2018-10-29/granules/CDIR-2018-10-29-AL-H-6/summary?api_key=DEMO_KEY

https://api.govinfo.gov/packages/CDIR-2018-10-29/granules/CDIR-2018-10-29-CA-H-33/summary?api_key=DEMO_KEY

https://api.govinfo.gov/packages/CDIR-2018-10-29/granules/CDIR-2018-10-29-ME-S-1/summary?api_key=DEMO_KEY
