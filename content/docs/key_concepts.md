# Key concepts
Drive is built for the cloud-first - therefore, the design decisions and API choices are influenced by the capabilities of cloud storage providers.

## Working with keys and not paths
Cloud providers like S3 and GCS have no concept of file paths or a directory structure - they work with keys. Keys can look like paths, i.e., `profile_photos/user_1.jpg`, but they are not resolved to construct file system paths.

For example, if you use `../../user_1.jpg` as the key name, the file won't be saved in two folders down the root of the storage directory. In fact, an exception will be thrown with this specific key name.

To mimic the behavior of cloud providers, we normalize and disallow certain characters in the key name and do not resolve paths like the Node.js path module.

Following is the list of normalizations and validations performed on a key before it is handed over to a driver.

### Pre-normalization phase

In the first phase, the keys are pre-normalized by applying the following transformations.

Multiple whitespaces are condensed into a single space.
- Backward slash (`\`) is replaced with a forward slash (`/`).
- Multiple slashes are replaced with one slash.

### Validating characters set

After the pre-normalization phase, the keys are validated to only include the following allowed characters. Otherwise, an `E_INVALID_KEY` exception will be thrown.

- Alphanumeric characters: `A-Z`, `a-z`, and `0-9`.
- Special characters: `-`, `_`, `/`, `!`, and `.`.
- Whitespaces.

### Preventing path traversal

The keys are further validated for sequences that can lead to [path traversal attacks](https://owasp.org/www-community/attacks/Path_Traversal), and the `E_PATH_TRAVERSAL_DETECTED` exception is raised.

### Post-normalization phase

After clearing the validation steps, a final set of normalizations is applied to the key.

- Current directory references `./` are removed.
- Leading and ending slashes are removed.
- Leading and ending dots are removed.

## Unified exceptions
To make the exception-handling experience consistent across all the drivers, we wrap errors inside generic exception classes. However, you can always access the underlying error thrown by a driver using the [`error.cause`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Error/cause) property.

Let's assume we want to read the contents of a non-existent file - the exception handling code may look like the following.

```ts
import { errors } from 'flydrive'

try {
  await drive.get(key)
} catch (error) {
  if (error instanceof errors.E_CANNOT_READ_FILE) {
    /**
     * Send a generic message in response
     */
    res.send('Unable to serve file')

    /**
     * Log the underlying error
     */
    console.error(error.cause.message)
  }
}
```

## Files visibility
Drive allows you to keep files `private` or `public` using the visibility option. Drivers under the hood translate the visibility option to provider-specific values.

The `public` files are accessible using their public URL and are served by the cloud provider directly. For example, You can access a public file via GCS using the `https://storage.googleapis.com/<bucket-name>/<key>` URL.

The `private` files are inaccessible publicly. You must generate a temporary (self-expired) URL to access the file or serve it from your application using the `disk.get` or `disk.getStream` methods.

## Directories as a second-grade citizen
Cloud storage services do not have a concept of directories and folders - they store files as key-value pairs. However, some cloud services allow you to emulate certain [directory-based operations](https://docs.aws.amazon.com/AmazonS3/latest/userguide/using-folders.html) using prefixes.

Therefore, the support for performing actions on a directory is limited. However, Drive does allow you to **list all** and **delete all** files matching a given prefix.
