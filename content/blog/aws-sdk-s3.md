---
title: "Interacting with the ECS S3 API using the aws-sdk/client-s3 package"
date: 2021-12-07T10:13:42Z
draft: false
tags: ["Node.js", "ECS", "S3", "aws-sdk", "client-s3"]
showToc: true
---

Recently at work, a customer raised a ticket about programmatically interacting with [UKCloud's Cloud Storage service](https://docs.ukcloud.com/articles/cloud-storage/cs-gs.html).

> UKCloud's Cloud Storage service is an object storage solution based on Dell EMC Elastic Cloud Storage (ECS). Access is via a RESTful API, which also provides support for Amazon's S3 API.
>
> <cite>UKCloud: Getting Started Guide for Cloud Storage [^1]</cite>

This ticket was interesting as the customer was using the [@aws-sdk/client-s3](https://docs.aws.amazon.com/AWSJavaScriptSDK/v3/latest/clients/client-s3/index.html) JavaScript package to upload images to the service. Prior to this ticket, I hadn't used this package before or have much experience with S3 object storage and programmatically interacting with it. In this blog post, I want to share my learnings and provide a couple of examples in Node.js for interacting with UKCloud's Cloud Storage service using this package.

> [NOTE]
>
> It is worth noting that the code examples in this post *should* work with any S3 object storage provider, just modify the endpoint and region!

## S3 Terminology

Before we get started, it's important to understand certain terminology around Amazon's S3 API.

### Objects

> An object is a file and any metadata that describes that file. Objects consist of object data and metadata. The metadata is a set of name-value pairs that describe the object. These pairs include some default metadata, such as the date last modified ... You can also specify custom metadata at the time that the object is stored.
>
> <cite>Amazon Web Services: What is Amazon S3? [^2]</cite>
>
> <cite>Amazon Web Services: Uploading, downloading, and working with objects in Amazon S3 [^3]</cite>

### Object Keys 🔑

An object key (or key name) is a unique identifier of an object.
> <cite>Amazon Web Services: Creating object key names [^4]</cite>

An example of an object key could be `Testing/Requirements.pdf` or `Accounting/Payslips.xls`. Note that the object keys are prefixed with a directory.

### Buckets

> A bucket is a container for objects... Every object is contained in a bucket.
>
> <cite>Amazon Web Services: What is Amazon S3? [^2]</cite>

A bucket has a flat structure with no actual concept of directories however, you can prefix object keys with (sub)directories to create a directory structure 📁

Now that's out of the way, lets begin interacting with the ECS S3 API using the @aws-sdk/client-s3 for JavaScript 😎

## Prerequisites

Before starting, you will need the following:

| Prerequisite  | Optional |
| ----- | --- |
| [Docker](https://www.docker.com/) installed on your system. | No |
| The S3 API endpoint for the object storage provider of your choice. In the examples I will be using [UKCloud's Cloud Storage endpoint](https://docs.ukcloud.com/articles/cloud-storage/cs-gs.html#api-endpoints): `https://cas.cor00005.ukcloud.com`. | No |
| The object storage provider's region name. In the examples I will be using: `cor00005`. | No |
| A bucket created with an object storage provider of your choice. | Yes - You can create one via the API however, for the examples I already had a bucket. |
| The object storage provider's equivalent of an `AccessKeyId` and `SecretAccessKey`. | No |

## Node.js Examples 👨‍💻

The following packages are used in these examples:

* [@aws-sdk/client-s3](https://www.npmjs.com/package/@aws-sdk/client-s3) - [Documentation](https://docs.aws.amazon.com/AWSJavaScriptSDK/v3/latest/clients/client-s3/index.html).

* [@aws-sdk/s3-request-presigner](https://www.npmjs.com/package/@aws-sdk/s3-request-presigner) - [Documentation](https://docs.aws.amazon.com/AWSJavaScriptSDK/v3/latest/modules/_aws_sdk_s3_request_presigner.html).

* [fs](https://nodejs.org/api/fs.html)

Begin by creating two files: `examples.mjs` and `dockerfile`. Then, paste the snippet below into the `dockerfile`:

```
FROM node:16-alpine
RUN mkdir -p /usr/src/app && chown -R node:node /usr/src/app
WORKDIR /usr/src/app
RUN npm install @aws-sdk/client-s3@3.43.0 @aws-sdk/s3-request-presigner@3.44.0
COPY --chown=node:node examples.mjs examples.mjs
USER node
CMD ["node", "examples.mjs"]
```

> [NOTE]
> Place the Node.js examples below into `examples.mjs`.

### Initialise the S3Client

Start by initialising the `S3Client` by importing the required package:

```javascript
import { S3Client } from '@aws-sdk/client-s3';

// Retrieve constants from environment variables
const S3_ENDPOINT = process.env.S3_ENDPOINT;
const REGION = process.env.REGION;
const BUCKET_NAME = process.env.BUCKET_NAME;

// Initialise client
const client = new S3Client({
    endpoint: S3_ENDPOINT,
    region: REGION,
});
```

### Sending Commands with the S3Client

Now, lets use the `S3Client` to create an object in a bucket. For this type of action use the `PutObjectCommand`:

#### Create an Object

```javascript
import { S3Client, PutObjectCommand } from '@aws-sdk/client-s3';

// ...

// Create an object (testObject.txt) prefixed with a directory (test-dir/) in a bucket
try {
    await client.send(
        new PutObjectCommand({
            Bucket: BUCKET_NAME,
            Key: 'test-dir/testObject.txt',
            Body: 'Uploaded test object.'
        })
    );
    console.log(`Successfully created object 'test-dir/testObject.txt' in the bucket '${BUCKET_NAME}'`);
} catch (err) {
    console.error(`An error occurred creating object 'test-dir/testObject.txt' in the bucket '${BUCKET_NAME}': ${err}`);
};
```

Lets see if it works! :smile: Build and run the container image:

```bash
docker build -t aws-sdk-s3:nodejs .
# The aws-sdk looks for the environment variables
# AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY automatically
docker run --rm --name aws-sdk-s3 \
    -e S3_ENDPOINT="https://cas.cor00005.ukcloud.com" \
    -e REGION="cor00005" \
    -e BUCKET_NAME="db-bucket" \
    -e AWS_ACCESS_KEY_ID="..." \
    -e AWS_SECRET_ACCESS_KEY="..." \
    aws-sdk-s3:nodejs

# Output
Successfully created object 'test-dir/testObject.txt' in the bucket 'db-bucket'
```

:scream: It worked! Congrats! You've just created your first object in a bucket! 😎

#### List and Get Objects

Another action is to list objects in a bucket and get them. Here is an example I wrote for that:

```javascript
import {
    S3Client,
    PutObjectCommand,
    ListObjectsCommand,
    GetObjectCommand
} from '@aws-sdk/client-s3';

// ...

// List of all objects (max 1000) in a bucket
try {
    var objects = await client.send(
        new ListObjectsCommand({ Bucket: BUCKET_NAME })
    );
} catch (err) {
    console.error(`An error occurred listing objects for the bucket '${BUCKET_NAME}': ${err}`);
};

// Iterate over objects in the bucket and perform GetObjectCommand
try {
    for (const [index, object] of objects.Contents.entries()) {
        console.log(`${index} - Getting object: ${object.Key}`);
        const resp = await client.send(
            new GetObjectCommand({
                Bucket: BUCKET_NAME,
                Key: object.Key
            })
        );
        console.log(`${index} - Object: ${object.Key}, ETag: ${resp.ETag}`);
    };
} catch (err) {
    console.error(`An error occurred getting objects for the bucket '${BUCKET_NAME}': ${err}`);
};
```

#### Upload and Download Objects

Uploading and downloading objects is done often, so below is examples I wrote for this:

```javascript
// ...
import fs from 'fs';

// ...

// Create a file to be upload
fs.writeFileSync('/usr/src/app/uploadfile.txt', 'Upload me!', (_) => {
    console.log("Successfully wrote file '/usr/src/app/uploadfile.txt'");
});

// Upload an object (file) to a bucket
try {
    // Load the contents of 'uploadfile.txt'
    fs.readFile('/usr/src/app/uploadfile.txt', 'utf8', async function (_, data) {
        await client.send(
            new PutObjectCommand({
                Bucket: BUCKET_NAME,
                Key: 'uploadfile.txt',
                Body: data
            })
        )
        console.log(`Successfully uploaded object '/usr/src/app/uploadfile.txt' to the bucket '${BUCKET_NAME}'`);
    });
} catch (err) {
    console.error(`An error occurred uploading object '/usr/src/app/uploadfile.txt' to the bucket '${BUCKET_NAME}': ${err}`);
};

// Download an object (file) from a bucket
try {
    const downloadFile = await client.send(
        new GetObjectCommand({
            Bucket: BUCKET_NAME,
            Key: 'test-dir/testObject.txt'
        })
    );
    // Create a write stream
    const writeStream = fs.createWriteStream('/usr/src/app/testObject.txt');
    // Pipe the object's body to the write stream
    downloadFile.Body.pipe(writeStream);
    console.log(`Successfully downloaded object 'testObject.txt' from the bucket '${BUCKET_NAME}'`);
} catch (err) {
    console.error(`An error occurred downloading object 'test-dir/testObject.txt' from the bucket '${BUCKET_NAME}': ${err}`);
};
```

#### Generate a Public URL for an Object

The final example I want to share is generating a public URL for an object so it can be downloaded. The example below generates a URL valid for one hour:

```javascript
// ...
import { getSignedUrl } from "@aws-sdk/s3-request-presigner";

/// ...

// Get a public URL for an object in a bucket
// Valid for 1 hour
try {
    const command = new GetObjectCommand({
        Bucket: BUCKET_NAME,
        Key: 'uploadfile.txt'
    });
    // Get public URL for the object
    const publicUrl = await getSignedUrl(client, command, { expiresIn: 3600 });
    console.log(`Successfully generated public URL for object 'test-dir/testObject.txt' in the bucket '${BUCKET_NAME}': ${publicUrl}`);
} catch (err) {
    console.error(`An error occurred generating public URL for object 'test-dir/testObject.txt' in the bucket '${BUCKET_NAME}': ${err}`);
};
```

The examples above and others that I wrote can be found on GitHub at: https://github.com/dbrennand/ecs-s3-examples/tree/nodejs/nodejs

I hope you learned something from this blog post! Until next time! :smile: :wave:

## References

[^1]: https://docs.ukcloud.com/articles/cloud-storage/cs-gs.html

[^2]: https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html

[^3]: https://docs.aws.amazon.com/AmazonS3/latest/userguide/uploading-downloading-objects.html

[^4]: https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-keys.html
