---
title: "Why Not Use Heroku for Your Ghost Blog"
date: 2019-06-26T21:18:08-04:00
draft: false
toc: false
images:
tags: 
  - ghost
  - heroku
---

I tried to set up a personal blog for free using [Heroku](https://heroku.com) and [Ghost](https://ghost.org). After some time trying to setup this, I've come to the conclusion that this really does not works as planned. The main problem is how Heroku works for our basic use case of a blog.

## How Heroku works

For our needs to host a blog, Heroku is just a service where you can host your website files. It follows these steps to serve your website:

- Right off the bat, your files will not be available to access.
- When someone access your website's url, Heroku will create a Dyno instance for your website.
- A Dyno instance means that your website will be built somewhere to become available to access.
- The build process can take a long time if your website is a more complex web app such as the case of Ghost that is a Node.js app.
- So when the first user hits the app, he will have to wait about 10 seconds to get any response. The following ones will not have to wait.
- After some time not being used, the Dyno will be destroyed. 
- When this happens, any file uploaded to that Dyno will be destroyed and when someone tries to access it again, it will have to rebuild your site once again.
- Databases do not follow this rule. Anything that was saved in these will be kept.

None of these points are a problem. This is just how the service works. And this is a good thing since they charge for the time that your Dyno stays alive. They offer a free tier plan where you can have up to 1000 hours per month of time consumed by a Dyno.

## How to surpass Heroku's ways

Since Heroku destroys everything when it does not need it anymore, how to keep images that I've uploaded to my blog? Just save them somewhere else. Our bloging plataform in question (Ghost), provides some 3rd party libraries that allow us to save an image through its UI to somewhere else. Unfortunately, the most common ones does not work properly.

Ghost call these libraries [storage adapters](https://docs.ghost.org/concepts/storage-adapters/). The following ones rouse my interest:

- [S3](https://github.com/spanishdict/ghost-s3-compat)
- [Google Drive](https://github.com/robincsamuel/ghost-google-drive)
- [Imgur](https://github.com/wrenth04/ghost-imgur)
- [Github](https://github.com/ifvictr/ghost-github)
- [Github Pages or Gitlabe Pages](https://github.com/zce/pages-store)

These are services that I was used to use, but most of these libraries do not work as expected or the services in question do not provide what we need.

For Google Drive and Github cases, the libraries just did not work. For the Github library, the author said that [it has some bugs that needs fixing for the latest version of Ghost](https://github.com/ifvictr/ghost-github/issues/11). The Google Drive one was just throwing an exception whenever I tried to upload an image. The Imgur one was kinda half-baked, it just uploaded the images as an anonymous, so in the end you would not have a nice collection with all your images somewhere to check.

The ones that worked where the S3 and Github Pages or Gitlab Pages. The S3 one worked flawless. The only downside is that S3 is a paid service. You can uses its storage service for 12 months, but after that you pay or lose access. Since I wanted a free solution, this was not an option. The other one also worked very well, but then came Github and Gitlab to bite me back. Github pages where giving me random 404s for the images I uploaded. Sometimes I could access an image, others don't. And for Gitlab the images just took some time to become available, something around 5 minutes. In this case Gitlab was a good solution, but since S3 show me that it would be nice to use a service where I could upload and have the image available instantly I decided to keep looking further for something else.

But I didn't. There is actually a last option that seems pretty similar to S3 and it is free: [Google Cloud Storage](https://cloud.google.com/storage/). There is also a [storage adapter for it](https://github.com/thombuchi/ghost-google-cloud-storage). In this case Gitlab would be the solution if the Google Cloud Storage does not work very well with this.

In the end this solves only part of the problem. Images are ok, but themes files are not. Ghost does not uses these storage adapters to save themes files. So when you upload a nice theme you found somewhere, Heroku will just wipe out later to turn your webiste in just a mess of text without style. This problem makes Heroku not a good option for a dynamic blog that needs a backend such as Ghost or even Wordpress. Only if these CMS provided a way to save all the files to an external storage.

## Why Heroku in the first place?

Because of the [one click deploy button](https://github.com/cobyism/ghost-on-heroku). You just click and you have your website. This link evens shows how to setup S3 to use along with it. But unfortunately this does not covers everything. Alos I knew about Heroku for some time, so I just thought that was worth the shot.

## Why Ghost?

In relation with Wordpress, that it is a very popular CMS, Ghost is more simple, elegant and have a nice post editor, perfect for someone that just want to access your blog from anywhere and start typing something.

## Conclusion

Dropped the idea and went for a static blog with [Hugo](https://gohugo.io/). Now I have to write the posts in a text editor using markdown instead of a WYSIWYG editor, but that works just fine. Now I do not have problems with images or other things, just commit the contents to my github repository and waits some minutes to go live. So, do not try to put a CMS or any simple web app that needs a backed in a service like Heroku. It was not made for this kinda of thing. Not for a free tier at least.