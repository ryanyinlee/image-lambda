# image-lambda

## How to use the Lambda

```js

let AWS = require('aws-sdk');

let S3 = new AWS.S3();

exports.handler = async (event) => {
    console.log(event.Records[0].s3.object);
    let manifest = await S3.getObject({ Bucket: 'myorangebucket', Key: 'images.json' }).promise();
    
    console.log(manifest);
    // TODO implement
    const response = {
        statusCode: 200,
        body: JSON.stringify({message: 'testing s3 put'}),
    };
    return response;
};

```

## Issues Encountered

Access Denied or Key Not Found after messing with permissions.

## Link to images.json

None available. Could not create or get one.

Was able to upload an image that should be public: https://myorangebucket.s3.us-west-2.amazonaws.com/images/birbeat.jpg


## Feature Tasks

- [ X ] Create an S3 Bucket with “open” read permissions, so that anyone can see the images/files in their browser
- [ X ] A user should be able to upload an image at any size, and update a dictionary of all images that have been uploaded so far
- [ X ] When an image is uploaded to your S3 bucket, it should trigger a Lambda function which must:
    - [ ] Download a file called “images.json” from the S3 Bucket if it exists
    - [ ] The images.json should be an array of objects, each representing an image. Create an empty array if this file is not present
    - [ ] Create a metadata object describing the image
    - [ ] Name, Size, Type, etc.
    - [ ] Append the data for this image to the array
    - [ ] Note: If the image is a duplicate name, update the object in the array, don’t just add it
    - [ ] Upload the images.json file back to the S3 bucket

**NOTE** If you setup your S3 Bucket to trigger your Lambda function on every file uploaded or modified, it will run that Lambda function every time that .json file is re-uploaded, putting you into an infinite loop. Be sure and set the event trigger to only run on files with image extensions as shown below.