# image-lambda

## How to use the Lambda

```js
let AWS = require('aws-sdk');

let S3 = new AWS.S3();

exports.handler = async (event) => {
    console.log(event.Records[0].s3.object);
    const fileName = event.Records[0].s3.object.key;
    const fileSize = event.Records[0].s3.object.size;
    const bucketName = event.Records[0].s3.bucket.name;
    
    console.log("BUCKET IS " + bucketName);
    
    let metaData = {
        name: fileName,
        size: fileSize,
        type: "image"
    };
    
    console.log("metaData is " + JSON.stringify(metaData));
    
    try {
        
    let manifestBody = await S3.getObject({ Bucket: bucketName, Key: 'images.json' }).promise();
    
    console.log(JSON.parse(manifestBody.Body.toString()));
    
    let manifestJSON = JSON.parse(manifestBody.Body);
    
    console.log(manifestJSON);
    
    
    manifestJSON.push(metaData);
    console.log("manifestJSON had metaData added.");
    
    let newManifest = await S3.putObject({ 
        Bucket: bucketName, 
        Key: 'images.json', 
        Body: JSON.stringify(manifestJSON),
    }).promise();
        
    console.log(newManifest);
        const response = {
        statusCode: 200,
        body: JSON.stringify({message: 'Image file logged.'}),
    };
    return response;
    } catch (e){
        console.log(e);
        
        let manifest = [metaData];
        
        let newManifest = await S3.putObject({ 
            Bucket: bucketName, 
            Key: 'images.json', 
            Body: JSON.stringify(manifest) 
            
        }).promise();
        return{
            message: 'New manifest generated.',
        }
    }

};
```

## Issues Encountered

Access Denied or Key Not Found after messing with permissions. 

UPDATE: 2/22/22: Jacob showed us how to fix the permissions.

There was an issue I encountered where I made the trigger less permissive. Making the prefix start with just images made the trigger continuously loop.



## Link to images.json

None available. Could not create or get one.

Was able to upload an image that should be public: https://myorangebucket.s3.us-west-2.amazonaws.com/images/birbeat.jpg

Update 2/2/22: Jacob gave us the info on how to bypass Access Denied: https://myorangebucket.s3.us-west-2.amazonaws.com/images.json


## Feature Tasks

- [ X ] Create an S3 Bucket with “open” read permissions, so that anyone can see the images/files in their browser
- [ X ] A user should be able to upload an image at any size, and update a dictionary of all images that have been uploaded so far
- [ X ] When an image is uploaded to your S3 bucket, it should trigger a Lambda function which must:
    - [ X ] Download a file called “images.json” from the S3 Bucket if it exists
    - [ X ] The images.json should be an array of objects, each representing an image. Create an empty array if this file is not present
    - [ X ] Create a metadata object describing the image
    - [ X ] Name, Size, Type, etc.
    - [ X ] Append the data for this image to the array
    - [ X ] Note: If the image is a duplicate name, update the object in the array, don’t just add it
    - [ X ] Upload the images.json file back to the S3 bucket

**NOTE** If you setup your S3 Bucket to trigger your Lambda function on every file uploaded or modified, it will run that Lambda function every time that .json file is re-uploaded, putting you into an infinite loop. Be sure and set the event trigger to only run on files with image extensions as shown below.