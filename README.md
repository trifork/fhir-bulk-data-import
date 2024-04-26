# fhir-bulk-data-import

A short writeup of how to get loads of data into a HAPI FHIR server setup. Guide assumes familiarity with [FHIR](hl7.org/fhir), [HAPI FHIR](https://github.com/hapifhir/hapi-fhir-jpaserver-starter), [Synthea](https://github.com/synthetichealth/synthea) & [Bulk Import](https://github.com/smart-on-fhir/bulk-import/blob/master/import-manifest.md)


# Steps
* Generate data using synthea having bulk export enabled.
* Generate the Parameters file using the parameters script below (the file format is bespoke to HAPI - see https://smilecdr.com/docs/bulk/fhir_bulk_import.html#fhir-bulk-import) by executing the script in the fhir output folder of synthea.
* Start the Bulk Data Hosting Server using the Python script below by executing the script in the fhir output folder of synthea.
* Start the HAPI FHIR Jpa starter with `bulk_import_enabled: true` and `auto_create_placeholder_reference_targets: true`.
* `POST` the Parameters file contents to HAPI FHIR at the `$import` endpoint and with the `HTTP Header` `Prefer: respond-async`.
* Profit $$$

# Script for generating the Parameters file:

```
import os
import json



def create_fhir_parameters(directory_path):
    fhir_parameters = {"resourceType": "Parameters", "parameter": []}

    

    fhir_parameters["parameter"].append({"name": "inputFormat", "valueString": "application/fhir+ndjson"})
  
    for filename in os.listdir(directory_path):
         if filename.endswith('ndjson') :
         
            entry = { "name": "input", "part": []}
            # Create a FHIR Parameter entry for each file
            type = filename.split(".")
            part1 = {
                "name": "type", "valueCode": type[0]}

            part2 = {"name": "url", "valueUri": 
                    host + filename
                
            }

            entry["part"].append(part1)
            entry["part"].append(part2)

            # Add the file entry to the Parameters object
            fhir_parameters["parameter"].append(entry)

    return fhir_parameters

# Example usage:
host = "http://localhost:8000/"
directory_path = "."
fhir_params = create_fhir_parameters(directory_path)

# Print the FHIR Parameters object
print(json.dumps(fhir_params, indent=2))

```

# Script for Bulk Data Hosting Server:

```
import http.server
import socketserver

PORT = 8000

class HttpRequestHandler(http.server.SimpleHTTPRequestHandler):
    extensions_map = {
        '': 'application/octet-stream',
        '.json': 'application/json',
        '.ndjson': 'application/json',
    }

with socketserver.TCPServer(("", PORT), HttpRequestHandler) as httpd:
    print("serving at port", PORT)
    httpd.serve_forever()

```
