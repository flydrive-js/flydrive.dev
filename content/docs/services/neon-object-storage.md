# Neon Object Storage

[Neon Object Storage](https://neon.com/docs/storage/overview) is an S3-compatible storage service. Therefore, you can use the [S3Driver](https://github.com/flydrive-js/core/blob/develop/drivers/s3/driver.ts) to manage files on Neon. Make sure to install the following peer dependencies in your project.

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
      accessKeyId: 'NEON_ACCESS_KEY_ID',
      secretAccessKey: 'NEON_SECRET_ACCESS_KEY',
    },

    // highlight-start
    /**
     * Copy and paste the endpoint and region from the "Storage" tab
     * of the "Connect" dialog in the Neon Console
     */
    endpoint: 'https://br-cool-darkness-a1b2c3d4.storage.c-1.us-east-2.aws.neon.tech',
    region: 'us-east-2',
    forcePathStyle: true,
    supportsACL: false,
    // highlight-end

    bucket: 'flydrive',
    visibility: 'private',
  })
)
```

You may pass all the options accepted by the [@aws-sdk/client-s3](https://www.npmjs.com/package/@aws-sdk/client-s3) package to the `S3Driver`. However, the following options must always be defined when using Neon.

<dl>

<dt>

supportsACL

</dt>

<dd>

Disable support for ACL. This is because Neon does not implement the commands to update ACL. As a result, the `disk.getVisibility()` method will not perform an API call and return the `visibility` value from the config.

Similarly, the `disk.setVisibility()` method will result in a `NOOP`.

</dd>

<dt>

forcePathStyle

</dt>

<dd>

Neon only supports path-style URLs (`<endpoint>/<bucket>/<key>`). Therefore, you must set `forcePathStyle` to `true`.

</dd>

<dt>

endpoint

</dt>

<dd>

Always define the `endpoint` of your Neon branch. The endpoint should not include the bucket name.

</dd>

<dt>

region

</dt>

<dd>

Specify the region of your Neon project. For example, `us-east-2`.

</dd>

<dt>

bucket

</dt>

<dd>

The `bucket` option defines the Neon bucket for managing files.

</dd>

</dl>

## Creating public URLs

When creating the bucket on Neon, set its access level to `public_read`. The access level can only be set when creating the bucket from the Neon Console, CLI, or API.

Files inside a `public_read` bucket are served from the endpoint. Therefore, you do not have to define the `cdnUrl` option.

```ts
const disk = new Disk(
  new S3Driver({
    endpoint: 'https://br-cool-darkness-a1b2c3d4.storage.c-1.us-east-2.aws.neon.tech',
    forcePathStyle: true,
    bucket: 'flydrive',
  })
)

const URL = await disk.getUrl('avatar.png')
console.log(URL) // https://br-cool-darkness-a1b2c3d4.storage.c-1.us-east-2.aws.neon.tech/flydrive/avatar.png
```

You may also create a public URL by defining a custom URL builder within the config. For example:

```ts
// title: Self-generating public URLs
const disk = new Disk(
  new S3Driver({
    bucket: 'flydrive',
    endpoint: 'https://br-cool-darkness-a1b2c3d4.storage.c-1.us-east-2.aws.neon.tech',
    forcePathStyle: true,
    // insert-start
    urlBuilder: {
      async generateURL(key, bucket, s3Client) {
        return `https://some-custom-url/files/${bucket}/${key}`
      },
    },
    // insert-end
  })
)

const URL = await disk.getUrl('avatar.png')
console.log(URL) // https://some-custom-url/files/flydrive/avatar.png
```

## Creating signed URLs

Signed URLs are created to provide time-based access to a private file hosted on Neon. For example:

```ts
const disk = new Disk(new S3Driver({}))

const signedURL = await disk.getSignedUrl('invoice.pdf', {
  expiresIn: '30mins',
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
  ResponseCacheControl: 'max-age=604800',
})
```

## Having issues?

### Use the correct credentials

When creating a credential on Neon, grant it the `storage:read` and `storage:write` scopes. Use the `token_id` as the `accessKeyId` and the `s3_secret_access_key` as the `secretAccessKey`.

### Requests fail with `ERR_TLS_CERT_ALTNAME_INVALID` or `NoSuchBucket`

Make sure the `forcePathStyle` option is set to `true`. Otherwise, the bucket name is added to the hostname, which Neon does not support.

### Uploads fail with `501 Not Implemented`

Make sure the `supportsACL` option is set to `false`. Otherwise, FlyDrive sends the ACL with every upload, which Neon does not support.
