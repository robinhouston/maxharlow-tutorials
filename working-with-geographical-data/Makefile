all: data/london-9.topo.json

clean:
	rm -f shp/* data/*

.PHONY: all clean


shp/statistical-gis-boundaries-london.zip:
	curl -fLo "$@" https://files.datapress.com/london/dataset/statistical-gis-boundary-files-london/2016-10-03T13:52:28/statistical-gis-boundaries-london.zip

shp/London_Borough_Excluding_MHW.shp: shp/statistical-gis-boundaries-london.zip
	cd shp \
	&& unzip -oj "$(<F)" statistical-gis-boundaries-london/ESRI/London_Borough_Excluding_MHW.* \
	&& touch London_Borough_Excluding_MHW.*

data/income-of-tax-payers.csv:
	curl -fLo "$@" https://files.datapress.com/london/dataset/average-income-tax-payers-borough/2016-04-05T08:55:06/income-of-tax-payers.csv

data/income-of-tax-payers.json: data/income-of-tax-payers.csv
	csv2json --input-encoding Latin1 -o "$@" "$<"

data/income-of-tax-payers.ndjson: data/income-of-tax-payers.json
	ndjson-split 'd' < "$<" > "$@" || { rm -f "$@"; false; }

data/london-1.geo.json: shp/London_Borough_Excluding_MHW.shp
	shp2json -o "$@" "$<"

data/london-2.geo.json: data/london-1.geo.json
	reproject \
		--use-spatialreference \
		--from 'EPSG:27700' \
		--to 'EPSG:4326' \
		"$<" > "$@" || { rm -f "$@"; false; }

data/london-3.geo.json: data/london-2.geo.json
	geoproject 'd3.geoRobinson().fitSize([1000, 800], d)' \
		-o "$@" "$<"

data/london-4.ndjson: data/london-3.geo.json
	ndjson-split 'd.features' \
		< "$<" > "$@" || { rm -f "$@"; false; }

data/london-5.ndjson: data/london-4.ndjson data/income-of-tax-payers.ndjson
	ndjson-join \
		'd.properties.GSS_CODE' \
		'd.Code' \
		$^ > "$@" || { rm -f "$@"; false; }

data/london-6.ndjson: data/london-5.ndjson
	ndjson-map \
		'd[0].properties = { code: d[0].properties.GSS_CODE, name: d[0].properties.NAME, incomeMedian: d[1]["Median £ - 2013-14"] }, d[0]' \
		< $< > "$@" || { rm -f "$@"; false; }

data/london-7.topo.json: data/london-6.ndjson
	geo2topo \
        --newline-delimited \
		boroughs=$< \
		-o "$@"

data/london-8.topo.json: data/london-7.topo.json
	toposimplify \
		--planar-area 2 \
		-o "$@" "$<"

data/london-9.topo.json: data/london-8.topo.json
	topoquantize 1e3 \
		-o "$@" "$<"
