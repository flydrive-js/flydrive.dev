# Google Cloud Storage

You may use the [GCS Driver](https://github.com/flydrive-js/core/blob/develop/drivers/gcs/driver.ts) to manage files using Google Cloud Storage. Make sure to install the following peer dependency in your project.

```sh
npm i @google-cloud/storage
```

Once done, you can create an instance of the GCS Driver and use it as follows.

```ts
import { Disk } from 'flydrive'
import { GCSDriver } from 'flydrive/drivers/gcs'

const disk = new Disk(
  new GCSDriver({
    credentials: 'GCS_KEY',
    visibility: 'public',
    bucket: 'GCS_BUCKET',
    usingUniformAcl: true,
  })
)
```

You can pass all the options accepted by the [@google-cloud/storage](https://googleapis.dev/nodejs/storage/latest/global.html#StorageOptions) package to the `GCSDriver` along with the following/required options.

<dl>

<dt>

bucket

</dt>

<dd>

The `bucket` option defines the GCS bucket to use for managing files.

</dd>

<dt>

visibility

</dt>

<dd>

The `visibility` option specifies the object's [predefinedAcl](https://googleapis.dev/nodejs/storage/latest/global.html#UploadOptions) when it is written to the GCS bucket.

- The `public` visibility sets the `predefinedAcl` to `publicRead`.
- The `private` visibility sets the `predefinedAcl` to `private`.

</dd>

<dt>

usingUniformAcl

</dt>

<dd>

Specify if the Bucket is using [uniform ACL](https://cloud.google.com/storage/docs/uniform-bucket-level-access). When enabled, you cannot change the visibility of a single file, and it will be inherited from the Bucket.

</dd>

<dt>

urlBuilder

</dt>

<dd>

Define a custom URL builder for creating public and signed URLs. [Learn more](#creating-public-urls)

</dd>

</dl>

## Using an existing Storage client

If you already have an instance of the `Storage` class from the `@google-cloud/storage` package. You may pass it directly to the Flydrive `GCSDriver` as follows.

```ts
import { GCSDriver } from 'flydrive/drivers/gcs'
import { Storage } from '@google-cloud/storage'

// highlight-start
const storage = new Storage({
  projectId: 'your-project-id',
  keyFilename: '/path/to/keyfile.json',
})
// highlight-end

const driver = new GCSDriver({
  // highlight-start
  storage: storage,
  // highlight-end
  bucket: 'GCS_BUCKET',
  visibility: 'private',
})
```

## Creating public URLs

Public URLs can be created for files uploaded to GCS with `public` visibility. By default, GCS will [automatically cache public objects](https://cloud.google.com/storage/docs/caching) and serve them via its CDN and hence you do not have to configure any CDN URL with Drive.

```ts
const disk = new Disk(
  new GCSDriver({
    credentials: 'GCS_KEY',
    visibility: 'public',
    bucket: 'GCS_BUCKET',
    usingUniformAcl: true,
  })
)

const URL = await disk.getUrl('avatar.png')
console.log(URL) // https://storage.googleapis.com/testing-drive/avatar.png
```

You may also self create a public URL by defining a custom URL builder within the config. For example:

```ts
// title: Self generating public URLs
const disk = new Disk(
  new GCSDriver({
    credentials: 'GCS_KEY',
    visibility: 'public',
    bucket: 'GCS_BUCKET',
    usingUniformAcl: true,
    // insert-start
    urlBuilder: {
      async generateURL(key, bucket, storage) {
        return `https://some-custom-url/files/${bucket}/${key}`
      },
    },
    // insert-end
  })
)

const URL = await disk.getUrl('avatar.png')
console.log(URL) // https://some-custom-url/files/testing-drive/avatar.png
```

## Creating signed URLs

Signed URLs are created to provide time-based access to a private file hosted on GCS. For example:

```ts
const disk = new Disk(new GCSDriver({}))

const signedURL = await disk.getSignedUrl('invoice.pdf', {
  expiresIn: '30mins',
})
```

At the time of generating the signed URL, you can pass one of the following options along with the options accepted by [getSignedUrl](https://googleapis.dev/nodejs/storage/latest/File.html#getSignedUrl) method.

```ts
await disk.getSignedUrl('invoice.pdf', {
  expiresIn: '30mins',
  contentType: 'application/pdf',
  contentDisposition: 'attachment',

  /**
   * Additional options applicable for GCS only
   */
  cname: 'https://cdn.example.com',
})
```
