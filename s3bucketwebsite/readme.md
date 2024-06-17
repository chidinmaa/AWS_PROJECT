### HOW TO CREATE A STATIC WEBSITE IN S3 BUCKET
- Navigate to s3 in your aws,to create an s3 bucket.s3 is a bucket storage  and the data stored in an s3 is called an object.
- In my previous project,I showed you how to create a bucket through CLI.Now i will be showing you how to create a bucket through the management console.
In the left navigation pane, choose Buckets.

Choose Create bucket.

The Create bucket page opens.

Under General configuration, view the AWS Region where your bucket will be created.

Under Bucket type, choose General purpose.

For Bucket name, enter a name for your bucket.

The bucket name must have a unique name.Be between 3 and 63 characters long.
Consist only of lowercase letters, numbers, dots (.), and hyphens (-). For best compatibility, we recommend that you avoid using dots (.) in bucket names, except for buckets that are used only for static website hosting.
Begin and end with a letter or number.
- I created a bucket called "chidi.scalagos"
- Go to properties of the bucket you created and enable static website hosting.
- copy the bucket website endpoint and paste it in your browser.
http://chidi.scalagos.s3-website-us-east-1.amazonaws.com

- It will bring out a 403 error.
- In permissions edit the block all public access and untick and save. This will enable the public to access your object in your object.
- paste in the bucket policy.
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": {
                "AWS": "*"
            },
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::chidi.scalagos/*"
        }
    ]
}

```
- Go to your objects and upload index.html file.Add the folder.
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Simple HTML Page</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 0;
            padding: 0;
            background-color: #f0f0f0;
        }
        .container {
            width: 80%;
            margin: 0 auto;
            padding: 20px;
            background-color: #fff;
            box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
            border-radius: 5px;
            margin-top: 50px;
        }
        h1 {
            text-align: center;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Welcome to My Simple HTML Page</h1>
        <p>This is a simple HTML page.</p>
    </div>
</body>
</html>
```
- You can also upload pictures from your file explorer.
https://s3.amazonaws.com/chidi.scalagos/cat.JPG

![alt text](<Images/html.JPG>)

My s3 bucket!


