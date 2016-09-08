# go-whosonfirst-libpostal

## Install

```
make bin
```

## wof-libpostal-server

```
$> wof-libpostal-server -options

Usage of wof-libpostal-server:
  -gracehttp.log
	Enable logging. (default true)
  -host string
    	The hostname to listen for requests on (default "localhost")
  -pidfile string
    	   Where to write a PID file for wof-libpostal-server. If empty the PID file will be written to wof-libpostal-server.pid in the current directory
  -port int
    	The port number to listen for requests on (default 8080)
```

_Note you will need to install the underlying [libpostal](https://github.com/openvenues/libpostal) C library yourself in order for `wof-libpostal-server` to work._

`wof-libpostal-server` exposes the following endpoint

### Endpoints

#### GET /debug/vars

This exposes all the usual Go [expvar]() debugging output along with the following additional properies:

* AvgParse
* AvgExpand
* ErrInput
* ErrRead
* ErrMarshal
* ErrUnmarshal
* ReqExpand
* ReqParse
* SuccessExpand
* SuccessParse

_Note: This endpoint is only available from the machine the server is running on._

#### POST /expand

```
curl -s -X POST http://localhost:8080/expand -d '{"query": "475 Sansome St San Francisco CA" }' | python -mjson.tool
[
    "475 sansome saint san francisco california",
    "475 sansome saint san francisco ca",
    "475 sansome street san francisco california",
    "475 sansome street san francisco ca"
]
```

#### POST /parse

```
curl -s -X POST http://localhost:8080/parse -d '{"query": "475 Sansome St San Francisco CA" }' | python -mjson.tool
[
    {
        "label": "house_number",
        "value": "475"
    },
    {
        "label": "road",
        "value": "sansome st"
    },
    {
        "label": "city",
        "value": "san francisco"
    },
    {
        "label": "state",
        "value": "ca"
    }
]
```

## wof-libpostal-crawl

```
$> wof-libpostal-crawl -options /path/to/whosonfirst-data-venue-us/data

Usage of wof-libpostal-crawl:
  -libpostal-host string
    		  The host for the libpostal endpoint
  -libpostal-port int
    		  The host for the libpostal port (default 8080)
  -output string
    	  Where to write output data (default "libpostal.csv")
  -processes int
    	     The number of concurrent processes to clone data with (default 16)
```

`wof-libpostal-crawl` traverses a directory containin Who's On First records and queries an instance of `wof-libpostal-server` with an address string derived from SimpleGeo (`sg:`) properties that can be used to populate `addr:` properties. For example, this:

```
./bin/wof-libpostal-crawl -libpostal-host localhost -processes 200 /usr/local/data/whosonfirst-data-venue-au/data/
parsed 697614 files in /usr/local/data/whosonfirst-data-venue-au/data/ in 314.434 seconds avg ttq: 836 ms
```

Produces a file that looks like this:

```
wof:id,sg:address,lp:results
336966001,343 Montague Rd West End QLD 4810,"[{""label"":""house_number"",""value"":""343""},{""label"":""road"",""value"":""montague rd""},{""label"":""city_district"",""value"":""west end""},{""label"":""state"",""value"":""qld""},{""label"":""postcode"",""value"":""4810""}]"
186121023,3/ 53 Smith St Kempsey NSW 2440,"[{""label"":""level"",""value"":""3""},{""label"":""house_number"",""value"":""/ 53""},{""label"":""road"",""value"":""smith st""},{""label"":""suburb"",""value"":""kempsey""},{""label"":""state"",""value"":""nsw""},{""label"":""postcode"",""value"":""2440""}]"
```

_As mentioned `wof-libpostal-crawl` is currently designed for use with Who's On First documents with specific SimpleGeo prefixed keys: `sg:address, sg:city, sg:province, sg:postcode`. It will eventually be adapted for other things._

## See also

* https://github.com/openvenues/libpostal
* https://github.com/openvenues/gopostal