# New JSON encoding for Haystack proposal

This is a proposal for a new JSON encoding format to the [Haystack](https://project-haystack.org/) standard.

Haystack data has its own type system. It has these granular types…

- String
- Number (with units)
- Boolean
- Uri
- Ref
- Def
- Date
- Time
- Date time
- Coord
- XStr
- Symbol

It also includes these collection types…

- List: an array
- Dict: a map
- Grid: a table

The default encoding used with these values is Zinc.

Here’s an example of a Zinc grid…

    ver:"3.0" projName:"test"
    dis dis:"Equip Name",equip,siteRef,installed
    "RTU-1",M,@153c-699a "HQ",2005-06-01
    "RTU-2",M,@153c-699a "HQ",1999-07-12

Zinc can also be encoded to a JSON format. For example…

    {
    		"meta": {"ver":"3.0", "projName":"test"},
    		"cols":[
    				{"name":"dis", "dis":"Equip Name"},
    				{"name":"equip"},
    				{"name":"siteRef"},
    				{"name":"installed"}
    		],
    		"rows":[
    				{"dis":"RTU-1", "equip":"m:", "siteRef":"r:153c-699a HQ", "installed":"d:2005-06-01"},
    				{"dis":"RTU-2", "equip":"m:", "siteRef":"r:153c-699a HQ", "installed":"d:999-07-12"}
    		]
    }

## Problems with Zinc

Web Developers are used to working with JSON. Both servers (Node) and browsers include native JSON parsing libraries. In order to to work with Zinc encoded data using the standard encoding, the browser’s JavaScript engine has to parse a lot of very large strings. Tests show that parsing JSON is faster than parsing large Zinc encoding strings in these environments.

Even the JSON version of Zinc still has a lot string parsing involved for the granular types. For example, in the above table a site ref is `r:153c-699a HA`. This string still has to be parsed to get the actual reference value. Therefore if we use standard or JSON encoding for Zinc, a client still has a lot of work involved in parsing strings.

As well as performance issues, there’s also a data accessibility issue. Since no primitive JSON values are being used, a consumer of any REST APIs has to have some form of client library. This prevents customers using a lot of the standard tools and libraries out there for working with this style of JSON. One example is using JSON schema to validate scalar values automatically.

Zinc always requires a sophisticated client library to parse and work with Haystack data. Not requiring a client library (or a far lighter one) would make it easier for Developers to work with Haystack data.

## Hayson

A new encoding format for Haystack related data is required that’s JSON based. It’s nicked named Hayson.

The goal of this format is…

- All types shouldn’t require additional parsing by a client beyond `JSON.parse(…)` where possible.
- It should be simple for a developer or system integrator to read and work with.
- Hayson should look just like standard JSON wherever it possibly can.
- High fidelity: there is no loss of data in the encoding. For instance…
  - A Hayson dict can never be confused by a client with a Hayson grid.
  - A Haystack Ref should never be accidentally confused as a string by the string starting with '@'.
- All JSON data is valid Haystack data.
  - A string is a haystack string, a boolean is a haystack boolean, an object a dict, an array a list etc.

The encoding capitalizes on the extremely fast native JSON parsers all modern web browsers have.

See the [specification](spec.md) for more information.

See the supported [schemas](schemas.md) for how to integrate Hayson with IDEs and web frameworks.

## Example

Here's a simple grid of sites encoded using Hayson...

    {
    	"_kind":"grid",
    	"meta":{
    			"ver":"3.0"
    	},
    	"cols":[
    			"id",
    			"area",
    			"dis",
    			"geoAddr",
    			"geoCoord",
    			"geoCountry",
    			"geoPostalCode",
    			"geoState",
    			"geoStreet",
    			"hq",
    			"metro",
    			"occupiedEnd",
    			"occupiedStart",
    			"primaryFunction",
    			"regionRef",
    			"site",
    			"store",
    			"storeNum",
    			"tz",
    			"weatherRef",
    			"yearBuilt",
    			"mod"
    	],
    	"rows":[
    			{
    				"id":{
    						"_kind":"ref",
    						"val":"p:demo:r:25aa2abd-c365ce5b",
    						"dis":"Headquarters"
    				},
    				"area":{
    						"_kind":"number",
    						"val":140797,
    						"unit":"ft²"
    				},
    				"dis":"Headquarters",
    				"geoAddr":"600 W Main St, Richmond, VA",
    				"geoCity":"Richmond",
    				"geoCoord":{
    						"_kind":"coord",
    						"lat":37.545826,
    						"lng":-77.449188
    				},
    				"geoCountry":"US",
    				"geoPostalCode":"23220",
    				"geoState":"VA",
    				"geoStreet":"600 W Main St",
    				"hq":{
    						"_kind":"marker"
    				},
    				"metro":"Richmond",
    				"occupiedEnd":{
    						"_kind":"time",
    						"val":"18:00:00"
    				},
    				"occupiedStart":{
    						"_kind":"time",
    						"val":"08:00:00"
    				},
    				"primaryFunction":"Office",
    				"regionRef":{
    						"_kind":"ref",
    						"val":"p:demo:r:25aa2abd-5c556aba",
    						"dis":"Richmond"
    				},
    				"site":{
    						"_kind":"marker"
    				},
    				"tz":"New_York",
    				"weatherRef":{
    						"_kind":"ref",
    						"val":"p:demo:r:25aa2abd-a02bf086",
    						"dis":"Richmond, VA"
    				},
    				"yearBuilt":1999,
    				"mod":{
    						"_kind":"dateTime",
    						"val":"2020-01-09T18:17:34.232Z",
    						"tz":"UTC"
    				}
    			},
    			{
    				"id":{
    						"_kind":"ref",
    						"val":"p:demo:r:25aa2abd-96516c18",
    						"dis":"Short Pump"
    				},
    				"area":{
    						"_kind":"number",
    						"val":17122,
    						"unit":"ft²"
    				},
    				"dis":"Short Pump",
    				"geoAddr":"11282 W Broad St, Richmond, VA",
    				"geoCity":"Glen Allen",
    				"geoCoord":{
    						"_kind":"coord",
    						"lat":37.650338,
    						"lng":-77.606105
    				},
    				"geoCountry":"US",
    				"geoPostalCode":"23060",
    				"geoState":"VA",
    				"geoStreet":"11282 W Broad St",
    				"metro":"Richmond",
    				"occupiedEnd":{
    						"_kind":"time",
    						"val":"21:00:00"
    				},
    				"occupiedStart":{
    						"_kind":"time",
    						"val":"10:00:00"
    				},
    				"primaryFunction":"Retail Store",
    				"regionRef":{
    						"_kind":"Ref",
    						"val":"p:demo:r:25aa2abd-5c556aba",
    						"dis":"Richmond"
    				},
    				"site":{
    						"_kind":"marker"
    				},
    				"store":{
    						"_kind":"marker"
    				},
    				"storeNum":3,
    				"tz":"New_York",
    				"weatherRef":{
    						"_kind":"ref",
    						"val":"p:demo:r:25aa2abd-a02bf086",
    						"dis":"Richmond, VA"
    				},
    				"yearBuilt":1999,
    				"mod":{
    						"_kind":"dateTime",
    						"val":"2020-01-09T18:17:34.323Z",
    						"tz":"UTC"
    				}
    			}
    	]
    }

## Conclusion

Using Hayson instead of Zinc for all server and client communication provides the following benefits…

- Less of a learning curve for Developers.
- Uses a globally accepted encoding format.
- Simple to understand.
- No additional client libraries required to work with the data.
- High performance. Tests show it’s between 10 and 5.5 times faster than parsing standard Zinc encoded data in a browser.
