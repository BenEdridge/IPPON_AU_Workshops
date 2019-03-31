# Building a Website for Production on the Cloud ☁️

![alt text](media/Workshop1/architecture_large.png "Architecture")

# Introduction

This workshop will guide you through a full production Cloud Deployment of your own personal blog including domain registration, configuration, deployment options, HTTPS, performance and standard development practices. The primary focus is not writing HTML, CSS or JavaScript but integrating multiple tools, pipelines and configurations to create a seamless experience. This is applicable not only to website design but many areas.

>There are only two hard things in Computer Science: cache invalidation and naming things.
>
>-- Phil Karlton

## What are we building?

![alt text](media/Workshop1/website.png "Screenshot of final website")

### Static website

You will be generating a static website with `hugo`, a tool written in Go. `hugo` sites are fully extensible, they can be swapped out entirely for another generator or just plain old HTML, CSS and JavaScript.

- Hugo gives us an easy way to write templates in `markdown`, style them, then generate HTML posts/pages from the markdown formatting. Markdown is a plain text authoring syntax to create documents like the current document you are viewing. This file is written in flavour of Markdown for github.com.
- `hugo` is part of the the JAM stack paradigm which gives us flexibility, open APIs and decoupling between content authoring, styling and deployment.

You will then use your domain name provider (namecheap.com) to point to a cached version of your website on the Cloudflare Content Delivery Network (CDN). This will result in global distribution of your website and improved load times. Your final website will be hosted on Google Cloud Platform, utilising Github with CircleCI for a Continuous Deployment/Integration process that pushes code through the Google Cloud SDK and uploads the resulting build files to Google Cloud Storage automatically.

# Prerequisites 
Complete this before the workshop if you can, it will save time! Some of these instructions may differ depending on operating system.

- Make sure you have a Credit/Debit Card you can use for Google Cloud sign up
- Registrations:
  - [NameCheap](https://namecheap.com) Domain provider or a free [.me](https://nc.me) domain!
  - [Cloudflare](https://cloudflare.com) CDN & Extra DNS provider
  - [Google Cloud Platform](https://console.cloud.google.com/freetrial)
    - Make sure you enable billing and add your card
  - [GitHub](https://github.com) (Hosting & building your website content)
  - [CircleCI](https://circleci.com) (Our CI/CD tool that integrates with github.com) make sure you link this to your github.com account
- Download and install:
  - [Hugo](https://gohugo.io/getting-started/installing) a static site generator with markdown templating 
  - [Google Cloud SDK](https://cloud.google.com/sdk/install) To access Google Storage and APIs
  - [Git](https://git-scm.com/downloads) (To manage github.com repositories and track code)

# Content
1. [Creating our Website Content](#💻-Creating-our-Website-Content)
2. [Google Cloud Platform - Cloud Hosting Provider](#☁️-Google-Cloud-Platform-Cloud-Hosting-Provider)
3. [Domain Setup and DNS config](#🌎-Domain-Setup-and-DNS-config)
1. [Cloudflare - Content Delivery Network (CDN)](#⚡-Cloudflare-Content-Delivery-Network-(CDN))
2. [Automatic Deployment Continued](#Automatic-Deployment-Continued)
3. [Extensions](#Extensions)

# 💻 Creating our Website Content

## Why do we want a static website?
- A static website is primarily HTML, CSS and JS and is usually fully interactive on the client side
- This may include routing, local state, animation and all manner of other cool things!
- However, we do not store state to databases, manage sessions or authorisations
- Sometimes the line is blurred between static and dynamic with new models of Serverless and other third party libraries

### To host and run a website we have a number of options:
- Run a server ourselves (A bit vintage but fun!)
- Upload your website content to another server managed elsewhere
- Use a fully managed solution such as Wordpress/Wix
- Use a static hosting solution (No server side scripting or code!)

## Getting started

1. Login to github.com and `fork` the workshop repository using the button on the right, this will copy the repository into your own github.com account (note any new updates from the original repository will be merged later)

![alt text](media/Workshop1/fork_repo.png "Forking")

1. Now run a `git clone <url>` using the repository you forked in your own github. This will download the repository to the directory you are in.

![alt text](media/Workshop1/clone_with_https.png "Clone")

1. Once cloned `cd` into the repository and run `hugo new site my-blog` where `my-blog` is a name of your choice this will create a new directory for your hugo project called `my-blog`
2. Find a theme you like! Currently we are using [hugo-theme-even](https://github.com/olOwOlo/hugo-theme-even) if you don't like this you can find another but it may require a different setup!
3. Open your blog directory eg `my-blog` and open the `themes` dir
4. To add a theme we will be using: `git clone https://github.com/olOwOlo/hugo-theme-even even` this will clone the repo into the `even` directory
5. Once cloned, copy `even/exampleSite/config.toml` (our site configuration) to the `my-blog` dir and replace the default `config.toml`
6. Still in `my-blog` dir create a new blog post by running `hugo new post/name_of_your_post.md`
7. Edit your post at: `content/post/name_of_your_post.md` If you don't know markdown look here: https://www.markdownguide.org/basic-syntax/

Add something like the following:
```
  1 ---
  2 title: "My awesome article"
  3 date: 2019-03-29T21:50:00+11:00
  4 draft: false
  5 ---
  6
  7 # Hello welcome to my blog
  8
  9 Thanks for joining
```

`Note: the "draft" = false`

1.  Now try and run your web server `hugo server -D` Your server will now be running at: http://localhost:1313
2.  Don't stop the server and make some changes to your post. You should see it hot reload, `config.toml` also supports hot reloading.
3.  Stop the server using ctrl + c
4.  Run `hugo` to build all your asset and files the the `public` dir
5.  Check the `my-blog/public` directory to see you all the assets/files/folders created from our previous command
6.  Files in `public` are now the self contained build and will be served from Google Cloud Storage in the next steps.
7.  Make sure you only upload content in `public`

## 📚 Resources
The Jamstack paradigm - https://jamstack.wtf  
A list of awesome static site generators - https://www.staticgen.com  
A list of awesome headless content management systems for the jamstack - https://headlesscms.org

# ☁️ Google Cloud Platform - Cloud Hosting Provider

![alt text](media/Workshop1/bucket_creation.png "Screenshot of final design")

## Basics

There are a number of ways to host our website. For this workshop we will be using Google Cloud Platform due to the ease of use and free credit/trial period provided. `Google Cloud Storage` is Google's offering for "bucket" storage (many Cloud offerings have a similar service) this is blob file storage with a selection of other features including metadata, permissions and redundancy. You can use it to store whatever you want.
  
To upload to GCS you can use the `gcloud` tool or the web interface. `gcloud` is a command line interface allowing us submit a number of powerful commands to the GCP APIs and manage infrastructure.

## It's GCP time

1. Login to the [Google Cloud Console](https://console.cloud.google.com) with your Google account
2. Make sure you have enabled billing and your credit/debit card is added otherwise you can't create a project or use the service
3. **Optional:** Set billing alerts for something like $10 so you don't waste the credit.
4. Create a new project for example: `my-blog`
5. Create a bucket under `Storage` with a name like: `my-blog`
6. Select `permissions` and update permissions allowing `allUsers` access. This will allow anyone access to this bucket and give the role `Storage Object Viewer` this means anyone can access the content of the bucket. Be careful what you upload from now on.

![alt text](media/Workshop1/add_Users.png "GCP permissions")

7. Make sure you have generated your website files using `hugo` 
8. Then upload all files inside the `public` directory to this bucket
9. Go back to bucket browser and find the public link to your file. It will be similar to: https://storage.googleapis.com/bens-blog/index.html where `my-blog` is your bucket name chosen before.
10. Add a suffix for the bucket eg. index.html and 404.html so it will be the default page

![alt text](media/Workshop1/public_access.png "Public Access")

11. Make sure everything is displayed correctly. You may need to change the config.toml file `baseURL` or set your permissions correctly on the bucket.

```
baseUrl = "https://storage.googleapis.com/my-blog/"
```

1.  Every time you change the `config.toml` file you must rebuild the blog using `hugo` and upload
2.  The config is only used for building the website and generating all these static html files

## Starting to get Automated

To start automation we should understand how to automatically build and upload our website content.

1. Make sure you have the `GCloud SDK` downloaded and test using `gcloud help`
2. Before running any `gcloud` tools you will need to login to the same Google account that you signed up to GCP
3.  Run: `gcloud auth login` this will open a browser window to login to Google.com
4. Once logged in make some changes to your basic website or add another blog post like before
5. Run `hugo` again to build everything into the output directory `public`
6. Now you can use `gcloud` to upload to our bucket without using the gui. Make sure you `cd` into the `my-blog` dir and then run the command below with your bucket name you created before.
   
`gsutil` is the Google Storage Utility and allows you to manage GCS this command will do a recursive sync of the `public` dir to the GCS bucket and upload all files.

 ```
 gsutil rsync -R public gs://bucket-name
 ```

Now navigate to your bucket again, have a look and see if your files are there, you can access your site. It should be available at: http://storage.googleapis.com/bucket-name/index.html

Well done, onto our domain setup now!

## 🔥 Extra:
- Keep exploring the GCP console
- Look at alternative static site generators
- Keep digging into `hugo`

# 🌎 Domain Setup and DNS config

Time to make use of that `.me` domain or another domain. We need to use the Domain Name System (DNS) to point our domain to the endpoint in Google Cloud Platform.

1. Login to [Google Search Console](https://search.google.com/search-console/welcome) with the **same** account you used to Signup for Google Cloud Platform. Here you will need to add your new fancy domain to the console so we can verify our ownership of the domain for GCP.

![alt text](media/Workshop1/domain_verify.png "Domain Verify")

1. Enter your domain name you have registered as a property
2. You will be presented with an option `Verify ownership via DNS record` copy that record so you can add it to namecheap.com DNS settings.
3. Log into namecheap.com go to `Advanced DNS` set up a new DNS record with the valid `TXT` record with `TTL = 1min` to confirm your ownership of the domain.
4. After make sure you remove any other strange DNS records like redirects or for `www`

```
NAME        TYPE     DATA
@           TXT      google-site-verification=12356abcdef....
```

You should also add a `CNAME` record which points your domain to Google. If you are wondering what the `@` is just means only domain (no subdomain like www or api) will point to Google Storage. If you put `test` it would end up being a subdomain like test.yourdomain.com

```
NAME        TYPE     DATA
@           CNAME    c.storage.googleapis.com
```

1. Now go back to the search console and try to re-verify your ownership, it may take a little while but should verify! This means you can now link your domain with the GCS storage bucket
2. Log back into Google Cloud Platform and try create a new bucket with the name of your domain eg. `yourdomain.com` if verification was successful you should be able to do this.
3.  Try it out, you may need to wait a little bit for the DNS settings to propagate again
4. If you get any errors like below you may have some config issues check your permissions, bucket name etc.
5. If all is well your website should now be resolved through yourdomain.com

```
<Error>
<Code> </Code>
<Message> </Message>
</Error>
```

## 🔥 Extra:
- Explore some of the other DNS types
- Multiple buckets can be created for different subdomains

# ⚡ Cloudflare - Content Delivery Network (CDN)

You have finished creating your website, it's up and running on your new domain but you want speed and reliability

You did a test on [cdnplanet](https://www.cdnplanet.com/tools/cdnperfcheck) and your friends in the Europe and Philippines are getting slow response times 😢
To speed up content delivery you will need to cache files closer to your end users. The server might be located in Australia but you can cache globally over a large network. A CDN will intelligently route traffic to those edge locations so users can reduce latency thus speeding up load times.

Make sure you modify the records in namecheap.com to match the required DNS settings by cloudflare.com

1. Login in to your cloudflare.com account and obtain the new nameservers
2. You will be prompted to enter in your domain you own
3. Once entered you will get a similar info page as below

![alt text](media/Workshop1/cloudflare_setup.png "Screenshot of final design")

4. Log back into namecheap.com and update your nameservers 
5. Under the `Domain` settings in the dashboard for your domain change the `Nameservers` setting to the nameservers for cloudflare (These might be different on your system, don't copy the above!)
6. This may take quite a while to complete so you may only have you Cloudflare setup after you have left this workshop
7. Verify the settings are correct and confirm
8. `Important` move onto the next section then come back come back here
9. Go back to cloudflare.com and check the verification
10. If you are verified retry your domain using [cdnplanet](https://www.cdnplanet.com/tools/cdnperfcheck)
11. You should see improved results!
12. You will also need to go to the `rules` tab and create a rule `yourdomain.com/*` and enable full caching of everything including HTML pages

## 🔥 Extra:
- Cloudflare has a huge amount of options both free and paid, explore all the tabs!

# 🏗️ Automatic Deployment Continued

We can make use of Github and TravisCI/CircleCI for our deployment toolkit. We will link the CI system with our repository on GitHub.

## Using CircleCI for auto deployment

1. Create a `.circleci` folder in the root directory of the repository
2. Inside this directory create the following content in a `config.yml` file where `blog-name` is the directory in your repo and `domain.com` is your chosen domain, or the standard bucket if you don't have the domain. The below is a configuration for CircleCI, notice the docker image for google cloud-sdk.

```
version: 2
jobs:
  build:
    docker:
      - image: google/cloud-sdk

    steps:
      - checkout

      - run: |
          echo $GCLOUD_SERVICE_KEY | gcloud auth activate-service-account --key-file=-
          gcloud --quiet config set project ${GOOGLE_PROJECT_ID}
          gsutil rsync -R blog-name/public gs://domain.com
```

1. Login to circleci.com using github and `Follow` the repository to give CircleCI access to it.
2. Make sure you are inside the repository 
3. Run `git status` to see status of files
4. Run `git add` to add all files that you need
5. Make sure you commit your changes `git commit -m "Finished dev"`
6. Push your changes using `git push`
7. This will push all changes you have commited to github.com
8. As long as the `.circleci/config.yml` is present then cirlceci.com will pick up the repository and start trying to run the above
9. Wait for a push, you will see a red failure this means circleci.com is not authorised to access your GCS bucket and upload content

![alt text](media/Workshop1/ci_build_example.png "Build Status")

### For further info:  
We will be authorizing CircleCI access to push code changes to our Google Cloud Platform bucket.
https://circleci.com/docs/2.0/google-auth/


1. Make sure you follow this: https://cloud.google.com/sdk/docs/authorizing#authorizing_with_a_service_account
2. Log back into GCP
3. Go to `Service Accounts` Tab https://console.cloud.google.com/iam-admin/serviceaccounts
4. Create a service account for circleci to access your github account

![alt text](media/Workshop1/service_account.png "Create a service account")

5. Choose a name and description like `circleci` and `allows access to my-bucket`
6. Select `permissions` as `Storage Object Admin`
7. Skip user access
8. `Create key` then download the `json` file, this will be used to populate the environment variables for the circleci config.

The content will be similar to below:

```json
{
  "type": "service_account",
  "project_id": "bens-site-fb",
  "private_key_id": "12345",
  "private_key": "-----BEGIN PRIVATE KEY-----",
  "client_email": "circleci@....com",
  "client_id": "1234",
  "auth_uri": "https://accounts.google.com/o/oauth2/auth",
  "token_uri": "https://oauth2.googleapis.com/token",
  "auth_provider_x509_cert_url": "https://www.googleapis.com/oauth2/v1/certs",
  "client_x509_cert_url": "https://www.googleapis.com/robot/v1/metadata/x509/circleci%40bens-site-fb.iam.gserviceaccount.com"
}
```

13. Run `gcloud auth activate-service-account --key-file=file.json` where `file.json` is the file you downloaded
14. You should get a successful result `Activated service account credentials for...`
15. Once you have a service account add these environment variables to your project in CircleCI. where `GCLOUD_SERVICE_KEY` is the content of the json and `GOOGLE_PROJECT_ID` is the name of your project in the Google Cloud Console.
   
```
GCLOUD_SERVICE_KEY
GOOGLE_PROJECT_ID
```

![alt text](media/Workshop1/env_setup.png "Env")

1. Once everything has been setup make some changes to your website code
   
```
hugo
git commit -m "A new message noting your changes"
git push
```

1. Now check CircleCI, if everything is setup correctly you should see a nice green tick and success!
2. If not you may have made a small mistake somewhere, check the Environment variables
3. You can re-run the `build` in the CircleCI website and check error messages to make sure everything is setup correctly
4.  Login to GCP and check the content of your bucket. Has it changed?
5.  Now we you to login to cloudflare.com to carry out cache invalidation and remove our old content from cache this is under the `cache` tab in cloudflare
6.  purge everything
7.  Once done do a test of your website and see if there are any performance improvements!

## 🔥 Extra:
- Try use gitlab.com and the gitlab CI/CD features
- Look at into Travis and Jenkins as well

## 📚 Resources
Docker - docker.com   
Atlassian - [CI/CD principles](https://www.atlassian.com/continuous-delivery/principles/continuous-integration-vs-delivery-vs-deployment)

# Extending your project

<details>
<summary>Receiving Email @yourcustomdomain.com</summary>
<p>

## ✉️ Email Setup

> Note: To enable email forwarding in we will need to change the nameservers back to namecheap.com nameservers thus losing our ability to use the cloudflare.com CDN + HTTPS

We can go back to namecheap.com and enable `email forwarding` which will forward emails to our account used to signup to namecheap. We can set a `catch-all` or just have certain emails forward. Emails being sent won't be sent from this address but if you want to set that up as well check the extras section below.

1. Go to domain list
2. Manage
3. Email Forwarding

Set something up like the following:

![alt text](media/Workshop1/email_forwarding.png "Architecture")

Depending on whether you set a catch-all or not you should be able to send an email to the relevant address: eg. `<anything>@yourdomain.com`

This will then be forwarded to the email used for namecheap.com

</p>
</details>

<details>
<summary>Sending Emails from yourname@yourdomain.com</summary>
<p>

</p>
</details>

- Utilising Serverless platforms
  - AWS Lambda
  - Google Cloud Functions
  - The Serverless framework https://serverless.com/
- Extra subdomains and features eg. help.yourdomain.com
- Switching to Amazon Web Services
- SEO
  - business.google.com
  - search.console.google.com
  - [Google Webmasters](https://support.google.com/webmasters/answer/7451184?hl=en)
  
# Acknowledgements
https://github.com/olOwOlo/hugo-theme-even
