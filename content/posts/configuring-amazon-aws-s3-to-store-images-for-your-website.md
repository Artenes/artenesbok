---
title: "Configuring Amazon Aws S3 to Store Images for Your Website"
date: 2019-06-25T19:12:55-04:00
draft: false
tags: 
  - AWS
  - S3
---

## Before starting

- Have an AWS (Amazon Web Services) account ([create one here](https://portal.aws.amazon.com/billing/signup#/start)).

## Things to note

- S3 (Simple Storage Service) is a paid service offered by Amazon.
- S3 is used to store any kind of data (images, videos, music and such).
- S3 has a free tier where you can store up to 5GB of data for one year, after this you have to pay to keep access. **There is not forver free plan for S3**.
- You can use an API to store the files or you can mannually do it by the S3 console in AWS' website.

## Creating an AWS user for your website

I am assuming that you will use some kind of CMS or website that already knows how to save and get images from S3. If that's the case, then you need to create an user in your AWS account for this website to be able to do this. **If you plan to just upload the images mannualy to S3 then you do not need any more users, so you can skip this step**.

- [Access the IAM (Identity and Access Management) console](https://console.aws.amazon.com/iam/home).
- Click in the "Add user" blue button.
- In the "User name" field give a name for the user of your website (e.g. MyWebsite).
- In the "Access Type" field, mark the "Programmatic access" checkbox.
- Click in the "Next: Permissions" blue button.
- Click in "Attach existing policies directly" option.
- In the search field search for "S3" (without the quotation marks).
- In the list of results click in the checkbox right before the "AmazonS3FullAccess" result.
- Click in the "Next: Tags" blue button.
- Click in the "Next: Review" blue button.
- Click in the "Create user" blue button.
- Click in the "Dowload .csv" button to dowload the data for your website user. "Access key ID" and "Secret access key" will be useful for your website to access your bucket programmatically.
- Click the "Close" button at the bottom right.
- Click in the name of the user you created in the list of users.
- Copy and paste the "User ARN" field's value to a text file. We will use this later.

## Creating a bucket

A bucket is just like a folder in your machine, a place to put files and more folders.

- [Access your S3 console](https://s3.console.aws.amazon.com/s3/home).
- Click in the blue button "Create bucket".
- Fill the field "Bucket name" with a name that makes sense for your bucket (e.g. myveryuniquewebsite).
- In the "Region" field, select a region that is closest to your region (e.g. US East (Ohio)).
- Click Create at the bottom left.

## Disabling blocking of public bucket policies

By default, S3 blocks any access to any of the files you upload to your buckets. We have to mannualy tell it to not block everything since our contents will be accessible by anyone in the web.

- Click in your bucket's name in the list of buckets.
- Click in the "Permissions" tab.
- Click in the "Edit" button at the right end of the "Block all public access" rectangle.
- Uncheck the first checkbox and check next two checkboxes, leaving the last two unchecked.
- Click in the blue "Save" button at the top right corner of the rectangle.
- A pop up will appear. Type the word "confirm" in the text field and hit the "Confirm" button.

## Defining Bucket Policies

In the previous step we've opened the access to our bucket. But now we have to strict it again so not anyone can try to upload files to our bucket. For this we will use Bucket Policies that makes use of its own language to describe for who we want give access to perform a certain operation in our bucket. In this case we want: 1. Let everyone see all the content in our bucket; 2. Only allow me and my website to put content in the bucket.

- While in the permissions tab of your bucket, click in the "Bucket Policy" sub-tab.
- Paste the following piece of Bucket Policy in the text field.

````json
{
    "Version": "2012-10-17",
    "Id": "Policy1561128789667",
    "Statement": [
        {
            "Sid": "AllowWebsite",
            "Effect": "Allow",
            "Principal": {
                "AWS": "PUT_YOUR_USER_HERE"
            },
            "Action": [
                "s3:ListBucket",
                "s3:DeleteObject",
                "s3:GetObject",
                "s3:PutObject"
            ],
            "Resource": [
                "PUT_YOUR_BUCKET_HERE",
                "PUT_YOUR_BUCKET_HERE/*"
            ]
        },
        {
            "Sid": "AllowViewer",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "PUT_YOUR_BUCKET_HERE/*"
        }
    ]
}
````
- Replace ``PUT_YOUR_BUCKET_HERE`` with the ARN (Amazon Resource Name) for your bucket. Just look for the value at the right of the title "Bucket policy editor" (e.g. arn:aws:s3:::myspecialuniquebucket). Pay attention to not delete the ``/*`` at the end of some of these.
- Replace ``PUT_YOUR_USER_HERE`` with the ARN of the user of your website. We already got this value in the first step, if you didn't, go back to the [IAM console](https://console.aws.amazon.com/iam/home) and get it (e.g. arn:aws:iam::388742887624:user/MyWebsite).
- Click in the blue "Save" button at the right top corner of where you pasted the code above.
- You will see a warning that "This bucket has public access". This is not a problem because that's out intention. The content here needs to be accessible to anyone. Just the upload needs to be restricted.

An explanation about the policy file above if you are interested. This file is telling S3 that we have two ``Statement``. One called ``AllowWebSite`` and other ``AllowViewer``. 

The ``AllowWebiste`` has an ``Effect`` to ``Allow`` our user (``Principal``) to perform a set of ``Action`` in the buckets with the names under ``Resource``. We have two here because for some actions we need to define that it needs to happen either on the whole bucket or just in the items inside the bucket (not the bucket itself).

The ``AllowViewer`` has an ``Effect`` to ``Allow`` any user (``*``) to perform the ``Action`` to view any file stored in our bucket with the name under ``Resource``.

## Conclusion

Now you can upload files directly in the S3 panel or give the "Access key ID" and "Secret access key" to your website so you can upload images through it (supposing your website already knows how to do this and you do not need to implement it yourself).

## Context

I was trying to put a [Ghost](https://ghost.org/) blog in [Heroku](https://heroku.com/). Since it does not support file storage I've decided to use S3. I found that it is very complicated to get the idea of the policies. But once done, you do not have to worry about it.