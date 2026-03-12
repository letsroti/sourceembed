# @letsroti/sourceembed

A small utility for storing raw text (typically embed JSON) in
**Cloudflare R2** and returning a **public URL**.

Useful for hosting embed payloads, temporary JSON data, or shareable
configuration files without building custom storage logic.

The library uploads text to an R2 bucket using the S3-compatible API,
generates a UUID-based filename, and returns a public URL using your
configured domain.

## Installation

Install the package using your preferred Node.js package manager.

``` bash
npm install @letsroti/sourceembed
```

or

``` bash
yarn add @letsroti/sourceembed
```

or

``` bash
pnpm add @letsroti/sourceembed
```

## Requirements

This package only needs two things.

### Cloudflare R2 Credentials

Valid **Cloudflare R2 API credentials** with permission to upload
objects to your bucket.

Required values:

-   Access Key ID
-   Secret Access Key
-   R2 S3-compatible endpoint URL

Example endpoint format:

    https://<account-id>.r2.cloudflarestorage.com

These credentials are used by the AWS S3 client to upload files to your
R2 bucket.

### Public Domain

You must also have a **public domain that serves files from your R2
bucket**.

Example:

    raw.example.com

Uploaded files will be returned as URLs in this format:

    https://raw.example.com/<uuid>.json

## Usage

### Import and Initialize

``` ts
import { SourceEmbed } from '@letsroti/sourceembed';

const sourceEmbed = new SourceEmbed({
  accessKeyId: process.env.R2_ACCESS_KEY_ID!,
  secretAccessKey: process.env.R2_ACCESS_KEY_SECRET!,
  endpoint: process.env.R2_ENDPOINT_URL!,
  bucket: process.env.R2_BUCKET!,
  publicDomain: process.env.R2_PUBLIC_DOMAIN!,
});
```

## Store a Single File

Uploads raw text to the configured R2 bucket and returns the generated
key and public URL.

``` ts
const embedData = {
  title: "Example",
  description: "Hello world"
};

const result = await sourceEmbed.store(
  JSON.stringify(embedData, null, 2)
);

console.log(result);
```

Example result:

``` ts
{
  key: "550e8400-e29b-41d4-a716-446655440000.json",
  url: "https://raw.example.com/550e8400-e29b-41d4-a716-446655440000.json"
}
```

## Store with Custom Extension

The file extension can be changed. Default extension is `json`.

``` ts
await sourceEmbed.store("Hello world", "txt");
```

Example URL:

    https://raw.example.com/<uuid>.txt

## Store Multiple Files

Uploads multiple texts in parallel.

``` ts
const contents = [
  JSON.stringify({ message: "one" }),
  JSON.stringify({ message: "two" }),
  JSON.stringify({ message: "three" })
];

const results = await sourceEmbed.storeMany(contents);
```

Example result:

``` ts
[
  { status: "fulfilled", value: { key: "...", url: "..." } },
  { status: "fulfilled", value: { key: "...", url: "..." } },
  { status: "rejected", reason: Error }
]
```

## Example Environment Variables

Store your Cloudflare R2 credentials as environment variables.

Example `.env` file:

```env
R2_ACCESS_KEY_ID=your_access_key_id
R2_ACCESS_KEY_SECRET=your_secret_access_key
R2_ENDPOINT_URL=https://your-account-id.r2.cloudflarestorage.com
R2_BUCKET=your_bucket_name
R2_PUBLIC_DOMAIN=raw.example.com
```

## License

MIT License
