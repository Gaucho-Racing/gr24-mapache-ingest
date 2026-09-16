# GR24 Mapache Ingest

> This service originally lived in the main Mapache monorepo. It was split into this repository as an archival snapshot while Mapache undergoes a complete v4 rewrite.

GR24 Mapache Ingest is the telemetry ingestion service used for Gaucho Racing's GR24 car. It is retained to preserve the vehicle's telemetry protocol, decoders, and storage model. It is not compatible with the current Mapache signal system.

## How it works

1. The service connects to the configured MQTT broker and subscribes to GR24 vehicle topics.
2. Topic-specific callbacks decode messages from the ACU, BCM, inverter, mobile client, pedal module, VDM, and four wheel modules.
3. A ping loop publishes health checks and records vehicle responses.
4. The decoded records are written directly to the configured SingleStore-compatible database.
5. The HTTP service exposes routes for retrieving the stored GR24 records.

The MQTT topics follow forms such as `gr24/{vehicle_id}/acu`, `gr24/{vehicle_id}/bcm`, and `gr24/{vehicle_id}/wheel/{position}`.

## Stored values

Unlike newer Mapache ingestion services, GR24 does not normalize telemetry into generic signal rows. Each source is saved to a dedicated relational model and table, including `ACU`, `BCM`, `Inverter`, `Mobile`, `Pedal`, `VDM`, `Wheel`, and `Ping`. The service creates and migrates these tables with GORM at startup.

This source-specific table layout is the main reason the service is archival rather than a current Mapache adapter.

## Running

The service is a Go application and can be built with:

```sh
go build ./...
```

Runtime configuration is supplied through environment variables for MQTT, the database, vehicle IDs, and the HTTP port. See `config/config.go` for the accepted settings.
