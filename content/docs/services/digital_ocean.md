# Digital Ocean Spaces
[Digital Ocean Spaces](https://www.digitalocean.com/products/spaces) is an S3-compatible storage service. Therefore, you can use the [S3Driver](https://github.com/flydrive-js/core/blob/develop/drivers/s3/driver.ts) to manage files on DO Spaces. Make sure to install the following peer dependencies in your project.

```sh
npm i @aws-sdk/s3-request-presigner @aws-sdk/client-s3
```

Once done, you can create an instance of the S3 Driver and use it as follows.

```ts
import { Disk } from 'flydrive'
import { S3Driver } from 'flydrive/drivers/s3'

const disk = new Disk(
  new S3Driver({
    credentials: {
      accessKeyId: 'AWS_ACCESS_KEY',
      secretAccessKey: 'AWS_ACCESS_SECRET',
    },

    // highlight-start
    endpoint: 'https://sgp1.digitaloceanspaces.com',
    region: 'sgp1',
    // highlight-end

    bucket: 'DO_BUCKET',
    visibility: 'private',
  })
)
```

You may pass all the options accepted by the [@aws-sdk/client-s3](https://www.npmjs.com/package/@aws-sdk/client-s3) package to the `S3Driver`. However, the following options must always be defined when using DO spaces.

<dl>

<dt>

endpoint

</dt>

<dd>

Make sure to always define the `endpoint` of the Digital Ocean Spaces service.

</dd>

<dt>

region

</dt>

<dd>

Define the `region` in which the bucket was created.

</dd>

<dt>

bucket

</dt>

<dd>

The `bucket` option defines the S3 bucket to use for managing files.

</dd>


</dl>

## Creating public URLs
Public URLs can be created for files uploaded to DO spaces with `public` visibility. The public URL can point to a CDN if you have configured the `cdnUrl` inside the driver config. Otherwise, it will fallback to the endpoint of your bucket region. For example:

```ts
// title: When CDN URL is configured
const disk = new Disk(
  new S3Driver({
    // highlight-start
    cdnUrl: 'https://testing-drive.sgp1.cdn.digitaloceanspaces.com',
    // highlight-end
    endpoint: 'https://sgp1.digitaloceanspaces.com',
    bucket: 'testing-drive'
  })
)

const URL = await disk.getUrl('avatar.png')
console.log(URL) // https://testing-drive.sgp1.cdn.digitaloceanspaces.com/avatar.png
```

```ts
// title: When CDN URL is not configured
const disk = new Disk(
  new S3Driver({
    // delete-start
    cdnUrl: 'https://testing-drive.sgp1.cdn.digitaloceanspaces.com',
    // delete-end
    endpoint: 'https://sgp1.digitaloceanspaces.com',
    bucket: 'testing-drive'
  })
)

const URL = await disk.getUrl('avatar.png')
console.log(URL) // https://sgp1.digitaloceanspaces.com/avatar.png
```

You may also self create a public URL by defining a custom URL builder within the config. For example:

```ts
// title: Self generating public URLs
const disk = new Disk(
  new S3Driver({
    bucket: 'testing-drive',
    endpoint: 'https://sgp1.digitaloceanspaces.com',
    // insert-start
    urlBuilder: {
      async generateURL(key, bucket, s3Client) {
        return `https://some-custom-url/files/${bucket}/${key}`
      }
    }
    // insert-end
  })
)

const URL = await disk.getUrl('avatar.png')
console.log(URL) // https://some-custom-url/files/testing-drive/avatar.png
```

## Creating signed URLs
Signed URLs are created to provide time-based access to a private file hosted on DO spaces. For example:

```ts
const disk = new Disk(
  new S3Driver({})
)

const signedURL = await disk.getSignedUrl('invoice.pdf', {
  expiresIn: '30mins'
})
```

At the time of generating the signed URL, you can pass one of the following options along with the options accepted by [GetObjectCommand](https://docs.aws.amazon.com/AWSJavaScriptSDK/v3/latest/client/s3/command/GetObjectCommand/) class.

```ts
await disk.getSignedUrl('invoice.pdf', {
  expiresIn: '30mins',
  contentType: 'application/pdf',
  contentDisposition: 'attachment',

  /**
   * Additional options applicable for S3 only
   */
  ResponseCacheControl: 'max-age=604800'
})
```
