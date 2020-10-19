# OpenStreetMap PBF Parser in Go

[![GoDoc](https://godoc.org/github.com/mattes/gosmparse?status.svg)](https://godoc.org/github.com/mattes/gosmparse) [![Build Status](https://travis-ci.org/thomersch/gosmparse.svg?branch=master)](https://travis-ci.org/thomersch/gosmparse)

gosmparse uses a callback driven API, which is stable ([Documentation](https://godoc.org/github.com/mattes/gosmparse)).

It has been designed with performance and maximum usage convenience in mind; on an Intel Core i7-6820HQ with NVMe flash it is able to process ~75 MB/s, so a planet file can be processed in under 10 minutes. If you find possible speed-ups or other improvements, let me know.


## Characteristics

* fast
* `panic`-free
* tested with different files from different sources/generators
* more than 80% test coverage and has benchmarks for all hot spots
* one dependency only: [gogo protobuf package](https://github.com/gogo/protobuf/proto) (a few more are used by tests and are included in the module)
* can read from any io.Reader (e.g. for parsing during download)
* supports history files

## Install

```
go get -u github.com/mattes/gosmparse
```

## Example Usage

```go
// Implement the gosmparser.OSMReader interface here.
// Streaming data will call those functions.
type dataHandler struct{}

func (d *dataHandler) ReadNode(n gosmparse.Node)         {}
func (d *dataHandler) ReadWay(w gosmparse.Way)           {}
func (d *dataHandler) ReadRelation(r gosmparse.Relation) {}

func ExampleNewDecoder() {
	r, err := os.Open("filename.pbf")
	if err != nil {
		panic(err)
	}
	dec := gosmparse.NewDecoder(r)
	// Parse will block until it is done or an error occurs.
	err = dec.Parse(&dataHandler{})
	if err != nil {
		panic(err)
	}
}
```

## Download & Parse

It is possible to parse during download, so you don't have to wait for a download to finish to be able to start the parsing/processing. You can simply use the standard Go `net/http` package and pass `resp.Body` to the decoder.

```go
resp, err := http.Get("http://download.geofabrik.de/europe/germany/bremen-latest.osm.pbf")
if err != nil {
	panic(err)
}
defer resp.Body.Close()
dec := gosmparse.NewDecoder(resp.Body)
err = dec.Parse(&dataHandler{})
if err != nil {
	panic(err)
}
```

## Did it break?

If you found a case, where gosmparse broke, please report it and provide the file that caused the failure.
