---
title: "Creating a Project Using Public Data for Fun and Profit: Part 3"
datePublished: Tue May 02 2023 15:37:20 GMT+0000 (Coordinated Universal Time)
cuid: clhapzjlu000009mk216ba2q9
slug: web-app-public-data-part-3
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1683300559417/c42590de-8606-45f7-ac2b-2fc4b4cb1c29.png
tags: seo, pwa, nextjs, google-analytics, vercel

---

This blog is part 3 of the "Creating a Project Using Public Data for Fun and Profit" series.  
Previous posts:

* [part 1](/blog/web-app-public-data-part-1)
    
* [part 2](/blog/web-app-public-data-part-2)
    

In this part, we will cover:

* Adding a favicon
    
* Adding a sitemap and robots.txt
    
* Adding PWA support
    
* Deploying our website
    
* Setting up a domain name
    
* Setting up email forwarding
    

All the code referenced in this post can be found in my repo [here](https://github.com/Zinbo/public-data-demo). As well, you can find a live version of the application we build [here](https://drivingpassrate.co.uk/).

# Adding the Finishing Touches

Congratulations on building your application! If you're planning on deploying it to the public, there are a few finishing touches that can take it to the next level of professionalism. In this section, we'll go over some easy but important tweaks that will make your app look even better.

## Adding an Icon

When you open your application at [`http://localhost:3000`](http://localhost:3000), you may notice that the favicon (the small icon that appears on the tab in your browser) is set to the Next.js logo. That's not very personalized, is it? We will change this to be the icon for our website.

There are plenty of places online where you can get free assets. One of my favorites is [flaticon.com](http://flaticon.com). Let's use the icon [here](https://www.flaticon.com/free-icon/pass_1633103?term=exam+pass&page=1&position=4&origin=search&related_id=1633103) for our website.

Before we get started, remember that favicons have a specific size: 32x32 pixels. So make sure to download the icon as a 32x32 PNG file. Once you have it, you'll need to convert it to a .ico file, which is what we need for our favicon. There are plenty of online tools for doing this, but one option is [here](https://image.online-convert.com/convert-to-ico). Just upload the PNG file and convert it to .ico.

Now that you have your `favicon.ico` file, it's time to replace the existing one in the `public` directory. This will ensure that your new icon is used when your app is deployed.

Once you've completed this step, head back to [http://localhost:3000](http://localhost:3000). You should see that the icon has changed on the tab.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1683300737558/a80afc6e-a643-4e33-a268-9cd8749b0f20.png align="center")

## Adding a Sitemap and robots.txt

Your website is almost ready to go live! However, before we deploy it, let's add a sitemap and a robots.txt file to improve its search engine optimization (SEO).  
Sitemaps help search engines understand the structure of your website and find all the relevant pages.  
The robots.txt tells search engines how to crawl your website.

To add these files to your website, we can use a package called `next-sitemap`. This package will automatically generate a sitemap and robots.txt for all of our static pages.

### Step 1: Add next-sitemap.config.js

First, we need to create a `next-sitemap.config.js` file in the root directory of our project. Copy the following code into the file:

```js
/** @type {import('next-sitemap').IConfig} */
const config = {
    siteUrl: process.env.SITE_URL || 'https://yourdomainname.com',
    generateRobotsTxt: true, // (optional)
    // ...other options
}
module.exports = config
```

This sets up the configuration for `next-sitemap`. Notice that `siteUrl` is set to [`https://yourdomainname.com`](https://yourdomainname.com). Don't worry, we will update this later.

### Step 2: Install next-sitemap

Next, let's add the `next-sitemap` package to our project by running the following command in our terminal:

```shell
npm install next-sitemap
```

### Step 3: Add postbuild Step

Once the package is installed, we need to add a `postbuild` step to our `package.json` file to generate the sitemap. Open the `package.json` file and add the following line after the `build` command:

```json
    "build": "next build",
    "postbuild": "next-sitemap",
```

Now, we can generate the sitemap by running the following command in our terminal:

```shell
npm run build
```

Make sure to have stopped your instance of `npm run dev` first.

Once the build is complete, you should see that 3 files have been generated in your `public` folder.

robots.txt:

```typescript
# *
User-agent: *
Allow: /

# Host
Host: https://yourdomainname.com

# Sitemaps
Sitemap: https://yourdomainname.com/sitemap.xml
```

sitemap.xml:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<sitemapindex xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
<sitemap><loc>https://yourdomainname.com/sitemap-0.xml</loc></sitemap>
</sitemapindex>
```

sitemap-0.xml:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9" xmlns:news="http://www.google.com/schemas/sitemap-news/0.9" xmlns:xhtml="http://www.w3.org/1999/xhtml" xmlns:mobile="http://www.google.com/schemas/sitemap-mobile/1.0" xmlns:image="http://www.google.com/schemas/sitemap-image/1.1" xmlns:video="http://www.google.com/schemas/sitemap-video/1.1">
  <url><loc>https://yourdomainname.com</loc><lastmod>2023-04-06T12:36:31.039Z</lastmod><changefreq>daily</changefreq><priority>0.7</priority></url>
  <url><loc>https://yourdomainname.com/cities</loc><lastmod>2023-04-06T12:36:31.039Z</lastmod><changefreq>daily</changefreq><priority>0.7</priority></url>
  <url><loc>https://yourdomainname.com/pass-rates</loc><lastmod>2023-04-06T12:36:31.039Z</lastmod><changefreq>daily</changefreq><priority>0.7</priority></url>
  <url><loc>https://yourdomainname.com/pass-rates/aberdeen</loc><lastmod>2023-04-06T12:36:31.039Z</lastmod><changefreq>daily</changefreq><priority>0.7</priority></url>
  <url><loc>https://yourdomainname.com/pass-rates/bangor</loc><lastmod>2023-04-06T12:36:31.039Z</lastmod><changefreq>daily</changefreq><priority>0.7</priority></url>
  <url><loc>https://yourdomainname.com/pass-rates/bath</loc><lastmod>2023-04-06T12:36:31.039Z</lastmod><changefreq>daily</changefreq><priority>0.7</priority></url>
  <url><loc>https://yourdomainname.com/pass-rates/birmingham</loc><lastmod>2023-04-06T12:36:31.039Z</lastmod><changefreq>daily</changefreq><priority>0.7</priority></url>
...
```

## How to Make Your App a Progressive Web App (PWA)

PWAs are a game-changer for providing a consistent experience across a variety of devices, and they allow users to install your web app on their device as if it were a native Android or iOS app. And the good news is that with Next.js, setting up your application as a PWA is quick and easy!  
You can read more about them [here](https://web.dev/learn/pwa/).

### Step 1: Install next-pwa

The first thing we'll do is install `next-pwa` with the following command:

```shell
npm install next-pwa
```

### Step 2: Update next.config.js

Next, we need to update our `next.config.js` file to use the `next-pwa` plugin. Here's the code you'll need:

```js
/** @type {import('next').NextConfig} */
const withPWA = require('next-pwa')({
  dest: 'public'
})

const nextConfig = withPWA({
  reactStrictMode: true,
  swcMinify: true,
})

module.exports = nextConfig
```

### Step 3: Create manifest.json

To make our PWA work, we need to create a `manifest.json` file. The easiest way to do this is to use a generator like [simicart](https://www.simicart.com/manifest-generator.html/). Make sure you download the icon at size 512 [here](https://www.flaticon.com/free-icon/pass_1633103?term=exam+pass&page=1&position=4&origin=search&related_id=1633103) as you'll need that for the generator.

Add the following properties to the Simicart manifest generator:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1683300757447/9a268dd6-ef24-4e3e-aabe-9976e536d3f7.png align="center")

Click "Generate Manifest" and you'll download a zip. Extract this zip to your `public` directory and rename `manifest.webmanifest` to `manifest.json`.  
Your `manifest.json` should look like this:

```json
{
  "theme_color": "#008000",
  "background_color": "#FFFFFF",
  "display": "standalone",
  "scope": "/",
  "start_url": "/",
  "name": "Best Driving Test Pass Rates Near Me",
  "short_name": "Driving Test Pass Rates",
  "description": "Give yourself the best opportunity to pass your driving test. Find the driving test centre that has the best pass rate near you. Find in locations such as Manchester, London, Birmingham, Newcastle, Leeds, Wales, Scotland, anywhere in the UK.",
  "icons": [
    {
      "src": "/icon-192x192.png",
      "sizes": "192x192",
      "type": "image/png"
    },
    {
      "src": "/icon-256x256.png",
      "sizes": "256x256",
      "type": "image/png"
    },
    {
      "src": "/icon-384x384.png",
      "sizes": "384x384",
      "type": "image/png"
    },
    {
      "src": "/icon-512x512.png",
      "sizes": "512x512",
      "type": "image/png"
    }
  ]
}
```

### Step 4: Add PWA tags to the Head element

Next, we need to add more tags to our `<Head>` element in the `_app.ts` file to provide more information on our PWA for different devices. Replace the `<Head>` element with the following:

```typescript
<Head>
  <title>Best Driving Test Pass Rates Near Me</title>
  <meta name="description"
        content="Give yourself the best opportunity to pass your driving test. Find the driving test centre that has the best pass rate near you. Find in locations such as Manchester, London, Birmingham, Newcastle, Leeds, Wales, Scotland, anywhere in the UK."/>
  <meta name='application-name' content='Best Driving Test Pass Rates Near Me' />
  <link rel="icon" href="/favicon.ico"/>

  {/* iOS */}
  <meta name="apple-mobile-web-app-capable" content="yes" />
  <meta name="apple-mobile-web-app-status-bar-style" content="default" />
  <meta name="apple-mobile-web-app-title" content="PWA App" />
  <meta name="format-detection" content="telephone=no" />

  {/* Android */}
  <meta name="mobile-web-app-capable" content="yes" />
  <meta name="theme-color" content="#008000" />

  <link rel="manifest" href="/manifest.json" />
  <link rel="shortcut icon" href="/favicon.ico" />
  <link rel="stylesheet" href="https://fonts.googleapis.com/css?family=Roboto:300,400,500" />

</Head>
```

Now, we can generate the PWA config by running the following command in our terminal:

```bash
npm run build
```

You'll see 2 new files in the `public` directory, `sw.js` and `workbox-<guid>.js`.  
These files are crucial for PWAs and you can learn more about them [here](https://developer.chrome.com/docs/workbox/).

Once the build process is complete, start up the app by running `npm run start`. Then, navigate to [`http://localhost:3000/`](http://localhost:3000/). You should now see an option to install the application.  

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1683300782825/3cad3312-f90f-4a85-9500-1e05605fbb2b.png align="center")

Just a quick heads up: I had to change the `display` property to `standalone` in manifest.json before Chrome allowed me to install the app as a PWA. You might need to do the same.

# Deploy the Application

We're now ready to deploy our application! We'll be using Vercel to deploy our Next.js application, as it's free for personal projects and is made by the creators of Next.js themselves.

## Deploying with Vercel

To deploy your application on [Vercel](https://vercel.com/), start by signing up on their website using your GitHub account. Once you're logged in, head to the [Dashboard](https://vercel.com/dashboard) and click the "Add New..." button followed by "Project". From there, click the "Import" button and select your project.  

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1683300799174/dfdb9a0e-81c5-4635-8f5f-d960c4501e24.png align="center")

After that, leave all the fields on the next screen as they are and click "Deploy". Vercel will now begin deploying your application, and you'll be able to see the progress of the deployment on your screen.  

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1683300807725/9999aac9-51d1-4215-a950-723dcc59d9c8.png align="center")

Once the deployment is finished, you can select your project and preview the landing page of your app. Click the "Visit" button to access your website, which now has a domain associated with it (e.g., [`public-data-demo.vercel.app`](http://public-data-demo.vercel.app)).  

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1683300813958/64621adb-007b-424c-a6f7-67367f0cd976.png align="center")

At this point, you now have a domain. For example, mine is [`public-data-demo.vercel.app`](http://public-data-demo.vercel.app). You can now go and change your `next-sitemap.config.js` file to point to this domain if you wish. Make sure to re-generate the robots.txt and the sitemap by running `npm run build` and check this into your repo.

If you would rather get your own domain without the [`vercel.app`](http://vercel.app) suffix then hang on till the next section!

### Vercel Analytics

One great feature of Vercel is its analytics. You can enable these for free by clicking the heartbeat button and then clicking "Enable".  

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1683300861196/229cbd36-cd55-46ab-b843-54472e2e02a1.png align="center")

This will give you valuable insights into your site's performance and an overall experience score based on multiple factors.  

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1683300869735/c7f1d74a-838d-4303-b06f-a718309bf4f2.png align="center")

A low score can negatively impact SEO with Google, so it's important to know how your site is performing and make any necessary improvements.

## Buy a Domain Name

You may want to have your own domain name that isn't associated with vercel. Domains can vary wildly in price depending on how popular the domain is.

I usually get my domains from [Google Domains](https://domains.google.com/) which is quick and easy to use. Say for example we wanted a domain related to the phrase `public-data-demo`. We can search for this in Google Domains and buy one for £10 a year:  

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1683300890033/481a16ea-6390-44b2-a251-4d5d3c4ce46b.png align="center")

We won't be walking through how to add a custom domain here, however I can assure you that is it very easy to do with Google Domains and Vercel. You can find a guide on how to do this [here](https://vercel.com/docs/concepts/projects/domains/add-a-domain).

You can now go and change your file to point to this domain if you wish.  
**Once your domain has been added make sure to update your** `next-sitemap.config.js` file to reflect your new domain name! Also, make sure to re-generate the robots.txt and the sitemap by running `npm run build` and push this into your repo. We'll get into why this is important in a later section.

### Email Forwarding

If you want to use a custom email address with your domain, but don't want to pay for Google Workspace, you can set up email forwarding for free.

To do this, go to your domain in Google Domains and navigate to the "Email" section. Click "Add email alias" and enter your desired email address at your domain, along with your personal email address that you want emails forwarded to. This will allow you to create an email address with your domain name (e.g., [hello@drivingpassrate.co.uk](mailto:hello@drivingpassrate.co.uk)) and forward it to your personal email.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1683300907189/3a482a9d-4b0a-4eae-9567-5ee17974e5df.png align="center")

# Conclusion

This completes part 3!  
Let's review what we've done here:

* Added a favicon
    
* Added a sitemap and robots.txt
    
* Made it a PWA
    
* Deployed with Vercel
    
* Got a domain name
    
* Set up email forwarding
    

You can find all the code used in this post in my repo [here](https://github.com/Zinbo/public-data-demo). As well, you can find a live version of the application we built [here](https://drivingpassrate.co.uk/).

Look out for the final part, where we'll be adding social media support, finding out how we can track website performance and SEO, and monetising our website!

Till next time!