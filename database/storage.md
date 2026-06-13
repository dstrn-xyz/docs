# storage

- [storage](#storage)
  - [introduction](#introduction)
  - [configuration](#configuration)
  - [obtaining a disk](#obtaining-a-disk)
  - [storing files](#storing-files)
    - [file uploads](#file-uploads)
  - [retrieving files](#retrieving-files)
  - [deleting files](#deleting-files)
  - [file management](#file-management)
  - [encryption](#encryption)
  - [disk space protection](#disk-space-protection)

<a name="introduction"></a>

## introduction

dframework provides a file system abstraction through the `Storage` facade. it allows you to easily read, write, and manage files across multiple configured "disks". the storage service automatically manages path traversal protection, symlinking public assets, concurrent operation queuing, and even disk space verification.

<a name="configuration"></a>

## configuration

the storage configuration is defined within the `app.storage.disks` configuration namespace. a typical configuration might include a `local` disk for private files, a `public` disk for assets meant to be publicly accessible, and a `secure` disk for encrypted files.

```javascript
storage: {
  disks: {
    local: {
      root: './storage/local', // private files, not web accessible
    },
    public: {
      root: './storage/public', // web accessible files
    },
    secure: {
      root: './storage/secure',
      encryptionKey: Env.value('APP_KEY'), // enables automatic aes-256-gcm encryption
    },
  },
},
```

dframework automatically creates a symbolic link from `public/storage` to `storage/public` upon boot, allowing files stored on the `public` disk to be served directly to the browser.

<a name="obtaining-a-disk"></a>

## obtaining a disk

you interact with the file system by calling the `disk` method on the `Storage` facade. if you do not specify a disk name, the framework defaults to the `public` disk.

```javascript
import { Storage } from 'dframework';

const disk = Storage.disk('local');
```

<a name="storing-files"></a>

## storing files

to write a file to the disk, use the `put` method. it accepts the relative file path as the first argument, and the file contents (as a string or buffer) as the second argument. the method will automatically create any necessary directories.

```javascript
await Storage.disk().put('avatars/1.jpg', imageBuffer);
await Storage.disk('local').put('exports/report.csv', 'id,name\n1,tarou');
```

<a name="file-uploads"></a>

### file uploads

the `put` method is fully aware of multipart file objects generated during http uploads. you can pass the uploaded file object directly to the `put` method, and the framework will stream it to its destination without requiring manual buffer extraction.

```javascript
// assuming `req.files.avatar` is a multiparty file upload
const path = await Storage.disk().put('avatars/user.jpg', req.files.avatar);
```

<a name="retrieving-files"></a>

## retrieving files

use the `get` method to read a file's contents. by default, it returns a raw `Buffer`. if you prefer a string, you can pass the desired encoding (like `utf8`) as the second argument.

```javascript
const raw = await Storage.disk('local').get('documents/contract.pdf');
const text = await Storage.disk('local').get('reports/summary.txt', 'utf8');
```

you can verify if a file exists on the disk using the `exists` method.

```javascript
if (await Storage.disk().exists('avatars/1.jpg')) {
  // file exists
}
```

if you need the absolute server path to a file on the disk, use the `path` method.

```javascript
const absolutePath = Storage.disk('local').path('exports/report.csv');
```

<a name="deleting-files"></a>

## deleting files

to remove a file, pass the path to the `delete` method. it returns `true` if the file was deleted, and `false` if the file did not exist.

```javascript
await Storage.disk().delete('avatars/1.jpg');
```

<a name="file-management"></a>

## file management

the framework provides convenience methods for moving and copying files within the same disk.

```javascript
// rename or move a file
await Storage.disk().move('temp/draft.txt', 'published/final.txt');

// duplicate a file
await Storage.disk().copy('templates/invoice.pdf', 'invoices/123.pdf');
```

<a name="encryption"></a>

## encryption

disks configured with an `encryptionKey` automatically encrypt on write and decrypt on read. the algorithm is aes-256-gcm. the key is derived from the provided string via sha256. the encryption key is never stored in the file. it lives only in the environment configuration (`APP_KEY`). the api is identical to unencrypted disks.

the on disk format is: 12 byte iv + 16 byte auth tag + ciphertext. the auth tag provides tamper detection. reads will fail if the file has been modified.

```javascript
// write, encrypted transparently
await Storage.disk('secure').put('keys/api-key.txt', 'my-secret-value');

// read, decrypted transparently
const value = await Storage.disk('secure').get('keys/api-key.txt', 'utf8');
```

<a name="disk-space-protection"></a>

## disk space protection

to prevent the application from crashing due to disk exhaustion, the storage service performs active health checks before writing data. before any `put` operation, the framework verifies that the target disk has sufficient space for the payload plus a one hundred megabyte safety buffer. if space is insufficient, the operation is safely aborted.
