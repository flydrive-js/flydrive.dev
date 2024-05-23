# Getting started

You may install Drive inside an existing Node.js project from the npm packages registry.

:::warning

Drive is an ESM-only package and will not work with the CommonJS module system.

:::

:::codegroup

```sh
// title: npm
npm i flydrive
```

```sh
// title: yarn
yarn add flydrive
```

```sh
// title: pnpm
pnpm add flydrive
```

:::

Once installed, you can import the [Disk class](./disk_api.md) and the driver you want to use to manage files. To keep things simple, we will use the [FSDriver](./services/fs.md) in this guide.

```ts
import { Disk } from 'flydrive'
import { FSDriver } from 'flydrive/drivers/fs'

const fsDriver = new FSDriver({
  location: new URL('./uploads', import.meta.url),
  visibility: 'public',
})

const disk = new Disk(fsDriver)
```

## CRUD operations
Once you have created an instance of the `Disk` class, you can use it to perform CRUD operations on a file. For example:

See also: [Disk API](./disk_api.md)

```ts
const key = 'hello.txt'
const contents = 'Hello world'

/**
 * Create a new file or override contents
 * of an existing file.
 */
await disk.put(key, contents)
```

To read files, you may use one of the following methods.

```ts
const key = 'hello.txt'

/**
 * Get contents as UTF-8 string
 */
const contents = await disk.get(key)

/**
 * Get contents as a Readable stream
 */
const stream = await disk.getStream(key)

/**
 * Get contents as Uint8Buffer
 */
const buffer = await disk.getArrayBuffer(key)
```

You can delete files using the `disk.delete` method.

```ts
const key = 'hello.txt'

await disk.delete(key)
```

## Using multiple drivers
You may use multiple drivers within a single application and/or switch between them depending upon the environment in which your app is running. 

Here's an example of managing drivers using environment variables. Feel free to tweak the example as per your application requirements.

See also: [Drive Manager](./drive_manager.md)

```ts
import { Disk } from 'flydrive'
import { FSDriver } from 'flydrive/drivers/fs'
import { GCSDriver } from 'flydrive/drivers/gcs'

/**
 * Step 1. Define a collection of drivers you plan to use
 */
const drivers = {
  fs: () => new FSDriver({
    location: new URL('./uploads', import.meta.url),
    visibility: 'public',
  }),
  gcs: () => new GCSDriver({
    region: 'sgp1',
    endpoint: 'https://sgp1.digitaloceanspaces.com',
    credentials: {
      accessKeyId: process.env.AWS_ACCESS_KEY_ID,
      secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY,
    },
    visibility: 'public',
  })
}

/**
 * Step 2. Pick the driver to use based upon the `DRIVE_DISK`
 * environment variable
 */
const driverToUse = drivers[process.env.DRIVE_DISK]

/**
 * Step 3. Create an instance of disk with the selected
 * driver
 */
export const disk = new Disk(driverToUse)
```

Once done, you can run your application with the `DRIVE_DISK` environment variable as follows.

```sh
# In development
DRIVE_DISK=fs node server.js

# In production
DRIVE_DISK=gcs node server.js
```
