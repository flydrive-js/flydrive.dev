# FlyDrive

FlyDrive is a file storage library for Node.js. It provides a unified API to interact with the local file system and cloud storage solutions like S3, R2, and GCS.

The primary goal of Drive is to read and write user-uploaded files without vendor lock-in on a single provider. For example, **during development, you may store files on your local file system** and **in production, you may switch to S3**.

Switching between the drivers is a simple configuration change. You do not have to modify your application's core logic.

## Official drivers

Following is the list of officially maintained drivers.

- [**S3Driver**](https://github.com/flydrive-js/core/blob/develop/drivers/s3/driver.ts) - The S3 driver can be used with [AWS S3](./services/s3.md), [Cloudflare R2](./services/r2.md) and the [Digital Ocean Spaces](./services/digital_ocean.md).

- [**GCSDriver**](https://github.com/flydrive-js/core/blob/develop/drivers/gcs/driver.ts) - The GCS driver is used to manage files on [Google Cloud Storage](./services/gcs.md).

- [**FSDriver**](https://github.com/flydrive-js/core/blob/develop/drivers/fs/driver.ts) - The FS driver keeps files on the local filesystem and uses the Node.js `fs` module under the hood.

## Why not Drive?

Drive serves a narrow use case, i.e., managing user-uploaded files with a unified API for all supported storage providers.

The constraints of a unified API prevent us from using the proprietary features of a specific vendor. For example:

- Drive cannot be used to create symlinks. While the local file system allows symlinks, cloud providers do not support this concept.

- You cannot watch the filesystem or assign Unix-style permissions to a file because cloud providers do not support this concept.

- Directories are treated as second-grade citizens. Cloud providers like S3 and GCS have no concept of directories - they operate as key-value stores.

## Why Drive?

Drive can be an excellent fit for applications that use cloud storage services to manage user-uploaded files. With Drive:

- You get a unified API for various storage services, including S3, R2, Digital Ocean spaces, GCS, and your local file system.
- Extensible API to add [custom storage drivers](./advanced/custom_drivers.md).
- [Unified exception handling](./key_concepts.md#unified-exceptions).
- Baked-in security primitives to prevent common attacks like [Path travesal](https://owasp.org/www-community/attacks/Path_Traversal).

## Next steps

- Read the [getting started guide](./getting_started.md) to install and use Drive.
- Learn about the [key concepts](./key_concepts.md) and design decisions of Drive.
- View [available API methods](./disk_api.md).

## Sponsors

::include{template="partials/sponsors"}
