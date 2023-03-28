pip install confluent-kafka

## Create the version 1

curl -s \
  -X POST \
  "http://localhost:8081/subjects/sensor-value/versions" \
  -H "Content-Type: application/vnd.schemaregistry.v1+json" \
  -d '{"schema": "{\"type\":\"record\",\"name\":\"sensor_sample\",\"fields\":[{\"name\":\"timestamp\",\"type\":\"long\",\"logicalType\":\"timestamp-millis\"},{\"name\":\"identifier\",\"type\":\"string\",\"logicalType\":\"uuid\"},{\"name\":\"value\",\"type\":\"long\"}]}"}' \
  | jq

## Create the version 2

Trying to create a new versio for the subject `sensor-value` by renaming the `timestamp` field as `ts`

curl -s \
  -X POST \
  "http://localhost:8081/subjects/sensor-value/versions" \
  -H "Content-Type: application/vnd.schemaregistry.v1+json" \
  -d '{"schema": "{\"type\":\"record\",\"name\":\"sensor_sample\",\"fields\":[{\"name\":\"ts\",\"type\":\"long\",\"logicalType\":\"timestamp-millis\"},{\"name\":\"identifier\",\"type\":\"string\",\"logicalType\":\"uuid\"},{\"name\":\"value\",\"type\":\"long\"}]}"}' \
  | jq

Returned:

```
{
  "error_code": 409,
  "message": "Schema being registered is incompatible with an earlier schema for subject \"{sensor-value}\""
}
```

Let's try to introduce a new field and try again.
The new field is `foo` of type `long`.

curl -s \
  -X POST \
  "http://localhost:8081/subjects/sensor-value/versions" \
  -H "Content-Type: application/vnd.schemaregistry.v1+json" \
  -d '{
   "schema":"{\"type\":\"record\",\"name\":\"sensor_sample\",\"fields\":[{\"name\":\"timestamp\",\"type\":\"long\",\"logicalType\":\"timestamp-millis\"},{\"name\":\"identifier\",\"type\":\"string\",\"logicalType\":\"uuid\"},{\"name\":\"value\",\"type\":\"long\",\"name\":\"foo\",\"type\":\"long\"}]}"
}' \
  | jq

Returned:

```
{
  "id": 2
}
```

*Adding a new field to the schema is backwards compatible, while renaming an existing fields breaks the backwards compatibility.*

