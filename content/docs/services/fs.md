# Local File system

You may use the [FS Driver](https://github.com/flydrive-js/core/blob/develop/drivers/gcs/driver.ts) to store files on the local file system. The visibility flag has no impact when using the FS driver, and it is up to your application to decide how to serve locally stored files.

You can create an instance of the FS Driver and use it as follows. The `location` property serves as the root directory from where files are read and stored. The value can be a `URL` or an absolute path to a directory.

```ts
import { Disk } from 'flydrive'
import { FSDriver } from 'flydrive/drivers/fs'

const disk = new Disk(
  new FSDriver({
    location: new URL('./uploads', import.meta.url),
  })
)
```

## URL builder

The FS Driver has no built-in capabilities to generate public or signed URLs. Therefore, using `disk.getUrl` or `disk.getSignedUrl` with the `fs` driver will result in an error.

However, you can register your custom implementation via the config object. For example:

```ts
import { SignedURLOptions } from 'flydrive/types'

const disk = new Disk(
  new FSDriver({
    location: new URL('./uploads', import.meta.url),
    urlBuilder: {
      generateURL(key: string, filePath: string) {
        return `https://yourapp.com/uploads/${key}`
      },

      generateSignedURL(key: string, filePath: string, options: SignedURLOptions) {
        /**
         * It is up to your application to decide how to create and verify
         * signed URLs. Do note this method can be async.
         */
        return `https://yourapp.com/uploads/${key}`
      },
    },
  })
)
```
