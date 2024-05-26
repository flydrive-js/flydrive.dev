# Disk API

Disk acts as an adapter between your application code and the underlying driver. It offers a unified API for performing file system operations, wraps errors inside [generic exception classes](./key_concepts.md#unified-exceptions) and [normalizes the key](./key_concepts.md#working-with-keys-and-not-paths) before handing it over to a driver.

![](./drive-architecture.jpeg)

## put

The `disk.put` method is used to create a new file or update an existing file. The method accepts the file key as the first parameter and its contents as the second parameter.

```ts
const disk = new Disk(driver)

const key = 'hello.txt'
const contents = 'hello world'

/**
 * Write file from raw contents
 */
await disk.put(key, contents)
```

| Param    | Type                             | Description                              |
| -------- | -------------------------------- | ---------------------------------------- |
| key      | `string`                         | Location of the file                     |
| contents | `string`, `Uint8Array`           | Contents for the file                    |
| options  | [`WriteOptions`](#write-options) | An optional metadata object for the file |

## putStream

The `disk.putStream` method works similarly to the `disk.put` method. However, it accepts the file contents as a Readable stream.

```ts
import { createReadStream } from 'node:fs'

const disk = new Disk(driver)

const key = 'hello.txt'
const readable = createReadStream('./some-file.txt')

/**
 * Write file from the readable stream
 */
await disk.putStream(key, readable)
```

| Param    | Type                             | Description                              |
| -------- | -------------------------------- | ---------------------------------------- |
| key      | `string`                         | Location of the file                     |
| contents | `Readable`                       | Contents for the file                    |
| options  | [`WriteOptions`](#write-options) | An optional metadata object for the file |

## get

The `disk.get` method is used to read the contents of a file as a `UTF-8` string. The method throws an exception if the file does not exist.

```ts
const disk = new Disk(driver)

const key = 'hello.txt'

/**
 * Get file contents as a string
 */
const contents = await disk.get(key)
console.log(contents)
```

## getStream

The `disk.getStream` method reads the contents of a file as a `Readable` stream. If the file does not exist, the method throws an exception.

```ts
import { createWriteStream } from 'node:fs'
import { pipeline } from 'node:stream/promises'

const disk = new Disk(driver)

const key = 'hello.txt'

/**
 * Create a readable stream for the file
 */
const readable = await disk.getStream(key)

/**
 * Write a readable stream to a writable stream
 */
await pipeline(readable, createWriteStream('./some-file.txt', readable))
```

## getArrayBuffer

The `disk.getArrayBuffer` method reads a file's contents as a `Uint8Array` stream. If the file does not exist, the method throws an exception.

```ts
const disk = new Disk(driver)

const key = 'hello.txt'

/**
 * Get Uint8Array
 */
const arrayBuffer = await disk.getStream(key)
console.log(new TextDecoder('utf-8').decode(arrayBuffer))
```

## delete

The `disk.delete` method deletes a file for the given key. The delete operation ignores non-existing files and does not throw an error.

```ts
const disk = new Disk(driver)

const key = 'hello.txt'
await disk.delete(key)
```

## deleteAll

The `disk.deleteAll` method deletes all the files matching the given prefix.

- In the case of the `fs` driver, this method will remove the matching directory and all its files using the `fs.rm` method.
- In the case of cloud providers like `gcs` and `s3`. This method will first fetch the list of files and delete them in parallel.

```ts
const disk = new Disk(driver)

const prefix = 'uploads/users/1'
await disk.deleteAll(prefix)
```

## copy

The `disk.copy` method is used to copy a file within the same bucket. This method will throw an error if the file to copy does not exist. It will also override the existing destination file (if one exists).

```ts
const disk = new Disk(driver)

const source = 'uploads/user_1.jpg'
const destination = 'uploads/avatars/user_1.jpg'

await disk.copy(source, destination)
```

| Param       | Type                             | Description                              |
| ----------- | -------------------------------- | ---------------------------------------- |
| source      | `string`                         | Location of the file to copy             |
| destination | `string`                         | Location of the destination file         |
| options     | [`WriteOptions`](#write-options) | An optional metadata object for the file |

## move

The `disk.move` method moves a file within the same bucket. This method will throw an error if the file that needs to be moved does not exist. Also, it will override the existing destination file (if one exists).

```ts
const disk = new Disk(driver)

const source = 'uploads/user_1.jpg'
const destination = 'uploads/avatars/user_1.jpg'

await disk.move(source, destination)
```

| Param       | Type                             | Description                              |
| ----------- | -------------------------------- | ---------------------------------------- |
| source      | `string`                         | Location of the file to move             |
| destination | `string`                         | Location of the destination file         |
| options     | [`WriteOptions`](#write-options) | An optional metadata object for the file |

## copyFromFs

The `disk.copyFromFs` method can copy a file from the local filesystem to a cloud provider. This method needs an absolute path for the file on the filesystem.

```ts
const disk = new Disk(driver)

const source = new URL('./invoice.pdf', import.meta.url)
const destination = 'clients/1/invoice.pdf'

await disk.copyFromFs(source, destination)
```

| Param       | Type                             | Description                              |
| ----------- | -------------------------------- | ---------------------------------------- |
| source      | `string`, `URL`                  | Path of the file to copy                 |
| destination | `string`                         | Location of the destination file         |
| options     | [`WriteOptions`](#write-options) | An optional metadata object for the file |

## moveFromFs

The `disk.moveFromFs` method can move a file from the local filesystem to a cloud provider. This method needs an absolute path for the file on the filesystem.

```ts
const disk = new Disk(driver)

const source = new URL('./invoice.pdf', import.meta.url)
const destination = 'clients/1/invoice.pdf'

await disk.moveFromFs(source, destination)
```

| Param       | Type                             | Description                              |
| ----------- | -------------------------------- | ---------------------------------------- |
| source      | `string`, `URL`                  | Path of the file to move                 |
| destination | `string`                         | Location of the destination file         |
| options     | [`WriteOptions`](#write-options) | An optional metadata object for the file |

## listAll

The `listAll` method can fetch a list of the files from the underlying storage provider. This method supports pagination and can optionally return a recursive list of files.

By default, this method will return top-level files and directory names along with a pagination token you may use to fetch more items, depending on the size of the bucket. For example:

```ts
const disk = new Disk(driver)

/**
 * Think of prefix as a sub-folder from which you
 * want to list files. Use `/` to fetch files
 * from the top level of the bucket.
 */
const prefix = 'uploads'

const response = await disk.listAll(prefix)

/**
 * Loop over the "objects" property, which is an "Iterator".
 * Each item in the list can be an instance of "DriveFile"
 * or "DriveDirectory"
 */
for (let item of response.objects) {
  if (item.isFile) {
    console.log(item.key)
  } else {
    console.log(item.prefix)
  }
}
```

The default mode is a great fit if you want to create a file explorer that lazily fetches files when a user opens a folder.

<!-- [Check out this example](), in which we use Unpoly, ShoeLoce, AdonisJS, and FlyDrive to build a file explorer. -->

### Recursive mode

In recursive mode, the `disk.listAll` method returns a collection of all the files (including files from sub-directories) as an Iterator. For example:

```ts
const disk = new Disk(driver)

/**
 * Think of prefix as a sub-folder from which you
 * want to list files. Use `/` to fetch files
 * from the top level of the bucket.
 */
const prefix = 'uploads'

const response = await disk.listAll(prefix, {
  // highlight-start
  recursive: true,
  // highlight-end
})

/**
 * Loop over the "objects" property, which is an "Iterator".
 * Each item in the list can be an instance of "DriveFile"
 */
for (let item of response.objects) {
  /**
   * Each item is always an instance of "DriveFile"
   */
  console.log(item.key)
}
```

### Pagination

In both the `default` and the `recursive` modes, you may perform pagination using the `paginationToken` and the `maxResults` options. The `paginationToken` is a cursor to fetch results after a given index.

In the following example, we perform pagination during an HTTP request. Feel free to adjust the code to fit the specifics of the framework you are using.

:::note

The `fs` driver does not support pagination; therefore, it returns the `paginationToken` as `undefined`. You can use the `undefined` value to disable the pagination feature or hide the "Load more" button in your application UI.

:::

```ts
router.get('/files', async ({ request }) => {
  const prefix = '/'
  const maxResults = 200
  const paginationToken = request.input('pagination_token')

  const response = await disk.listAll(prefix, {
    // highlight-start
    maxResults,
    paginationToken,
    recursive: true,
    // highlight-end
  })

  return {
    /**
     * "response.paginationToken" is shared with the client so that
     * they can send it back in the next request
     */
    nextToken: response.paginationToken,
    files: Array.from(response.objects),
  }
})
```

## getMetaData

The `disk.getMetaData` method is used to get the metadata of a file. The return value includes the following properties.

- `contentType`: The content type of the file. In the case of the `fs` driver, the file extension is used to infer the content type.
- `contentLength`: The size of the file in bytes.
- `etag`: Unique etag for the file. It could be used to cache the file.
- `lastModified`: A JavaScript Date object representing the last modified date of the file.

```ts
const disk = new Disk(driver)

const key = 'users/1/avatar.png'

const metaData = await disk.getMetaData(key)
console.log(metaData)
```

## exists

The `disk.exists` method can be used to check if a file exists or not. The method returns a boolean value.

```ts
const disk = new Disk(driver)

const key = 'hello.txt'

if (await disk.exists(key)) {
  await disk.get(key)
}
```

## getVisibility

The `disk.getVisibility` returns the visibility of the file.

- In the case of the `fs` driver, the value from the initial config is returned.
- In the case of cloud providers, we fetch the object ACL from their API and normalize the return value to [Drive visibility](./key_concepts.md#files-visibility).

```ts
const disk = new Disk(driver)

const key = 'hello.txt'
const visibility = await disk.getVisibility(key)

console.log(visibility)
```

## setVisibility

You may use the `disk.setVisibility` method to change the visibility of a file.

- In the case of the `fs` driver, this operation will return in a NOOP.
- The cloud providers will update the object ACL by making an API call.

```ts
const disk = new Disk(driver)
const key = 'hello.txt'

/**
 * Update visibility to public
 */
await disk.setVisibility(key, 'public')
```

## getUrl

The `disk.getUrl` method returns the public URL of a file.

This method does not check if the file exists or if the file has public visibility. So if needed, please perform these checks manually before generating and sharing the file's URL.

:::note

Make sure to also read the URL generation section of the service you are using to manage file uploads.

:::

```ts
const disk = new Disk(driver)
const key = 'hello.txt'

const url = await disk.getUrl(key)
console.log(url)
```

## getSignedUrl

The `disk.getSignedUrl` method returns a temporary signed URL of a file. This URL could be used to access a private file for a limited duration.

This method does not check if the file exists or if the file has public visibility. So if needed, please perform these checks manually before generating and sharing the file's URL.

:::note

Make sure to also read the URL generation section of the service you are using to manage file uploads.

:::

```ts
const disk = new Disk(driver)
const key = 'hello.txt'

const url = await disk.getSignedUrl(key, {
  expiresIn: '30 mins',
})
console.log(url)
```

The `getSignedUrl` method accepts the following arguments as the second parameter.

| Option             | Type               | Description                                                                           |
| ------------------ | ------------------ | ------------------------------------------------------------------------------------- |
| expiresIn          | `string`, `number` | The duration after which the URL will expire. Defaults to `30 mins`                   |
| contentType        | `string`           | Define the value of `Content-type` header set at the file of serving the file.        |
| contentDisposition | `string`           | Define the value of `Content-Disposition` header set at the file of serving the file. |

## Write options

Following is the list of options accepted as a third parameter by the `disk.put`, `disk.putStream`, `disk.copy`, `disk.move`, `disk.copyFromFs`, and `disk.moveFromFs` methods. For example:

```ts
/**
 * Define write options when creating a
 * new file.
 */
await disk.put(key, contents, {
  visibility: 'private',
  contentType: 'image/png',
})

/**
 * Define write options when moving a file
 */
await disk.move(source, destination, {
  visibility: 'private',
  contentType: 'image/png',
})
```

The drivers will translate these values to cloud provider-specific options.

<dl>

<dt>

visibility

</dt>

<dd>

Define the [visibility](./key_concepts.md#files-visibility) of the file. If not defined, the visibility will be inherited from the driver config.

- The `fs` driver ignores this option.
- The `gcs` driver will set the `predefinedAcl` option if the bucket is not using [uniform ACL](./services/gcs.md).

</dd>

<dt>

contentType

</dt>

<dd>

Define the `Content-type` header for the file. If not defined, it will be computed by the cloud service automatically.

- The `fs` driver ignores this option.
- The `gcs` driver will set the `metadata.cacheControl` option.

</dd>

<dt>

contentLanguage

</dt>

<dd>

Define the `Content-Language` header for the file. If not defined, the cloud service will compute it automatically.

- The `fs` driver ignores this option.
- The `gcs` driver also ignores this option.

</dd>

<dt>

contentEncoding

</dt>

<dd>

Define the `Content-Encoding` header for the file. If not defined, the cloud service will compute it automatically.

- The `fs` driver ignores this option.
- The `gcs` driver will set the `metadata.contentEncoding` option.

</dd>

<dt>

contentDisposition

</dt>

<dd>

Define the `Content-Disposition` header for the file. If not defined, it will be computed by the cloud service automatically.

- The `fs` driver ignores this option.
- The `gcs` driver also ignores this option.

</dd>

<dt>

cacheControl

</dt>

<dd>

Define the `Cache-Control` header for the file. If not defined, the cloud service will compute it automatically.

- The `fs` driver ignores this option.
- The `gcs` driver will set the `metadata.cacheControl` option.

</dd>

<dt>

contentLength

</dt>

<dd>

Define the file `Content-Length` header. If it is not defined, the cloud service will compute it automatically.

- The `fs` driver ignores this option.
- The `gcs` driver also ignores this option.

</dd>

</dl>
