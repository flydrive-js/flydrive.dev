# File API

You can create an instance of the [DriveFile](https://github.com/flydrive-js/core/blob/main/src/driver_file.ts) class using the `disk.file` method. A File acts as a pointer and can perform read-only operations on a given key.

In the following example, we create an instance of the `DriveFile` class and then use the `file.get` method to read its contents.

```ts
const disk = new Disk(driver)

const key = 'users/1/avatar.png'

/**
 * Create an instance of the file. Creating an
 * instance does not load the file contents
 */
const userAvatar = disk.file(key)

/**
 * Explicitly read file contents
 */
const contents = await userAvatar.get()
```

All the read-only methods from the [Disk](./disk_api.md) class are also available in the `DriveFile` class.

```ts
const file = disk.file(key)

/**
 * Get file contents as a string
 */
await file.get()

/**
 * Get file contents as a Readable stream
 */
await file.getStream()

/**
 * Get file contents as a Uint8Array
 */
await file.getArrayBuffer()

/**
 * Get file metadata
 */
await file.getMetaData()

/**
 * Check if the file exists within the storage
 * layer
 */
await file.exists()
```

## Working with the file snapshot

You can create a snapshot of a file using the `file.toSnapshot` method and persist it inside the database. The snapshot includes the file key and [its metadata](./disk_api.md#getmetadata). Snapshots can be a great way to list files in a UI without retrieving their metadata from the cloud provider.

```ts
const file = disk.file(key)
const snapshot = await file.toSnapshot()

console.log(snapshot)
/**
{
  key: 'users/1/avatar.png',
  contentType: 'image/png',
  contentLength: 31400,
  etag: 'W/"b-18f0eb15abf"',
  lastModified: '2024-04-24T06:01:09.065Z'
}
*/
```

Later, you can create an instance of the file from the snapshot using the `disk.fromSnapshot` method.

The file instance created using the `fromSnapshot` method caches the file metadata. However, to re-fetch fresh metadata, you must create the file instance using the `disk.file` method.

```ts
// title: Uses snapshot metadata
const file = disk.fromSnapshot(snapshot)
const metaData = await file.getMetaData()
```

```ts
// title: Fetches metadata from the storage provider
const file = disk.file(snapshot.key)
const metaData = await file.getMetaData()
```
