# Creating a custom driver

Creating a custom FlyDriver driver is as simple as creating a new JavaScript class that implements the [DriverContract](https://github.com/flydrive-js/core/blob/develop/src/types.ts#L72) interface. Your custom implementation can live anywhere, be it your application's source code or an NPM package.

```ts
import { Readable } from 'node:stream'
import { DriveFile, DriveDirectory } from 'flydrive'
import type {
  WriteOptions,
  DriverContract,
  ObjectMetaData,
  ObjectVisibility,
  SignedURLOptions,
} from 'flydrive/types'

export class AzureDriver implements DriverContract {
  /**
   * Return a boolean value indicating if the file exists
   * or not.
   */
  async exists(key: string): Promise<boolean> {}

  /**
   * Return the file contents as a UTF-8 string. Throw an exception
   * if the file is missing.
   */
  async get(key: string): Promise<string> {}

  /**
   * Return the file contents as a Readable stream. Throw an exception
   * if the file is missing.
   */
  async getStream(key: string): Promise<Readable> {}

  /**
   * Return the file contents as a Uint8Array. Throw an exception
   * if the file is missing.
   */
  async getArrayBuffer(key: string): Promise<ArrayBuffer> {}

  /**
   * Return metadata of the file. Throw an exception
   * if the file is missing.
   */
  async getMetaData(key: string): Promise<ObjectMetaData> {}

  /**
   * Return visibility of the file. Infer visibility from the initial
   * config, when the driver does not support the concept of visibility.
   */
  async getVisibility(key: string): Promise<ObjectVisibility> {}

  /**
   * Return the public URL of the file. Throw an exception when the driver
   * does not support generating URLs.
   */
  async getUrl(key: string): Promise<string> {}

  /**
   * Return the signed URL to serve a private file. Throw exception
   * when the driver does not support generating URLs.
   */
  async getSignedUrl(key: string, options?: SignedURLOptions): Promise<string> {}

  /**
   * Update the visibility of the file. Result in a NOOP
   * when the driver does not support the concept of
   * visibility.
   */
  async setVisibility(key: string, visibility: ObjectVisibility): Promise<void> {}

  /**
   * Create a new file or update an existing file. The contents
   * will be a UTF-8 string or "Uint8Array".
   */
  async put(key: string, contents: string | Uint8Array, options?: WriteOptions): Promise<void> {}

  /**
   * Create a new file or update an existing file. The contents
   * will be a Readable stream.
   */
  async putStream(key: string, contents: Readable, options?: WriteOptions): Promise<void> {}

  /**
   * Copy the existing file to the destination. Make sure the new file
   * has the same visibility as the existing file. It might require
   * manually fetching the visibility of the "source" file.
   */
  async copy(source: string, destination: string, options?: WriteOptions): Promise<void> {}

  /**
   * Move the existing file to the destination. Make sure the new file
   * has the same visibility as the existing file. It might require
   * manually fetching the visibility of the "source" file.
   */
  async move(source: string, destination: string, options?: WriteOptions): Promise<void> {}

  /**
   * Delete an existing file. Do not throw an error if the
   * file is already missing
   */
  async delete(key: string): Promise<void> {}

  /**
   * Delete all files inside a folder. Do not throw an error
   * if the folder does not exist or is empty.
   */
  async deleteAll(prefix: string): Promise<void> {}

  /**
   * List all files from a given folder or the root of the storage.
   * Do not throw an error if the request folder does not exist.
   */
  async listAll(
    prefix: string,
    options?: {
      recursive?: boolean
      paginationToken?: string
    }
  ): Promise<{
    paginationToken?: string
    objects: Iterable<DriveFile | DriveDirectory>
  }> {}
}
```

## Listing files

When listing files, you must use a `paginationToken` to fetch files in small chunks. If your driver implementation does not support pagination, you must return an `undefined` value for the `paginationToken` property.

The `objects` property is an Iterator that lazily constructs `DriveFile` and `DriveDirectory` instances. Make sure to consult existing [drivers implementation](https://github.com/flydrive-js/core/blob/develop/drivers/gcs/driver.ts#L446-L455) to understand how this iterator is created.

## Do not normalize keys

The Driver implementation should not [normalize keys](../key_concepts.md#working-with-keys-and-not-paths) as they receive already normalized keys from the [Disk](https://github.com/flydrive-js/core/blob/develop/src/disk.ts#L121) class.

## Do not wrap exceptions

The Driver implementation should not use [unified exceptions](../key_concepts.md#unified-exceptions) as the exceptions thrown by a driver are wrapped inside unified exceptions by the [Disk](https://github.com/flydrive-js/core/blob/develop/src/disk.ts#L125) class.
