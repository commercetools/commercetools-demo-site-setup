[START HERE](#necessary-accounts)

# Template and Documentation to set up a demo webshop with the commercetools platform

This README describes how to set up a sales demo as easy as possible. 

For goals vs. non-goals see the [Contribution Guide](CONTRIBUTE.md). 

You can start by downloading this repository as a ZIP file. 

## Necessary Accounts

(both free for the purpose of demo applications) 

  * [set up a personal account on the commercetools platform](https://mc.commercetools.com/). 
    If your company already has an "Organization" in the commercetools platform, ask a colleague to invite you. 
    Otherwise you will get a fresh personal organization on signup, wich is not eligible for partner terms if you are a partner.
  * [create an account at Heroku.com to temporarily host your SUNRISE demo web frontend (if wanted)](https://signup.heroku.com/). 
    Heroku is a service that lets you run server applications as "dynos" with own web address. 
    It's free for up to five dynos. If a free dyno is not used, it's put into sleep and restarted the next time someone is using it. 

## Tools and Software Implementations mentioned in this Tutorial: 

 * The CTP reference webshop ["SUNRISE" in Java](https://github.com/commercetools/commercetools-sunrise-java-starter) Java or [PHP](https://github.com/sphereio/commercetools-sunrise-php)
 * The [Heroku Toolbelt](https://toolbelt.heroku.com/) helps you in checking out and changing the shop frontend you created.
 * The [SUNRISE demo dataset](https://github.com/commercetools/commercetools-sunrise-data) provides sample store content if you do not want to generate customer specific content or only want to generate customer specific content in some areas like the product catalog.  

To understand and transform customer sample here's a selection of useful tools: 

 * a good XML and / or JSON viewer (if you're not in an IDE anyways)
 * CSV based product import:
   * Excel or LibreOffice to work manually with CSV files
   * (Mac or Linux only, 3rd party:) [CSVKit to analyze, clean up etc. CSV files](https://github.com/wireservice/csvkit)
   * [CSV mapper to bring the file into the right format](https://github.com/sphereio/csv-mapper/)
   * [CTP CSV Product Sync / Import](https://github.com/sphereio/sphere-node-product-csv-sync)
   * [Import Category Trees into CTP](https://github.com/sphereio/sphere-category-sync) via the [commercetools CLI](https://github.com/sphereio/sphere-node-cli)
 * JSON based product import:
   * [CTP product importer - part of the commercetools / sphere CLI](https://github.com/sphereio/sphere-node-cli)
   * [Import Category Trees into CTP](https://github.com/sphereio/sphere-category-sync) via the [commercetools CLI](https://github.com/sphereio/sphere-node-cli)
   
> All CTP Import / Export command-line tools can be used via the [IMPEX](https://impex.commercetools.com) web interface if you don't want to install them locally. 
  
## Phase 1: Analysis
 
> Question #1: Do you want to manually type in sample categories and products or rather import a data feed you received? 
 
Although typing in seems awkward, it can often be the much more efficient choice, especially when it comes to
product types (=the data model) and the categories. 
If you have not received sample data it's clear anyways: Take a coffee, open
[the Merchant Center](https://mc.commercetools.com/) and create the shop's world from scratch. 
 
If you have received sample data (often some product CSV feed or a specialized XML format), the first step is to get
an idea of the content, structure and quality: 
 
  1. Content: What's in? Is it enough for a demo to be sensible? (e.g. no description texts and no images -> demo will look bad -> type data yourself). Some random Tips:
   * CSV: Opening in a spreadsheet application can give a good first impression
   * CSV, mac & unix only: CSVkit's `csvstat` command generates very helpful statistics on column types, value distribution, value density etc.
  2. Structure: 
   * What field means what? 
   * Is there a concept of product variants? How are they identified?
   * Is it possible to create absolute paths to the live images? 
   * How is the price data structured? Net or Gross? VAT included in price or not? 
   * ... 
  3. Quality: Is it good enough for a visual demo (e.g. most fields empty? -> type data yourself)? 
 
Result: you know whether to try importing automatically or rather taking a subset of the catalog and typing that in. 
 
## Phase 2: Commercetools Setup / Signup

> note: settings have been migrated to the [Merchant Center](https://mc.commercetools.com/), too. 
  
  1. Log in, go to "Organization Settings", choose or create the Organization (e.g. a special org for your demo sites).
  2. Navigate to "Manage Projects" and create a new one. Please choose a key that makes clear it's a demo or playground by appending `-demo-1` or something similar. 
    * Leads to less confusion on commercetools' side about what this is
    * Does not "burn" the real project name key for the hopefully acquired customer. Keys cannot be changed and cannot be re-used.
  3. Don't add the sample dataset (it does only work well with specific language / country setups)
  4. Navigate to "Settings" and:
    * in Tab "International": configure just one language, country, currency and zone (it's not likely that you will import multilanguage, multichannel data in a sales demo and you can still do that later if you want)
    * in Tab "Taxes": add a Tax Category named "default", give that one Rate named "default" and set "included in price" if your prices are gross prices.
    * in Tab "Shipping Methods": Add a Shipping Method with the `name` of a carrier the prospect currently offers (or more). Set it as default. 
      * Add one Zone Rate
        * Add one Price with a "free above" setting (the structure has three levels of nesting)
  5. Navigate to "Developers" and
    * in Tab "Product Types", create a product type named "default", e.g. named "Standard Product"
      * for simple demos, it's easier to use a big "catch all attributes" product type definition. Better spend your time to model that one correctly than in differentiating types that then all just have plain String fields.  
      * (do the attribute modeling later)
    * Find the Tab "API Clients". You will need the Client ID and Client Secret later on. E.g. directly put them into the `.sphere-project-credentials` file of this project
     
## Phase 3: Fire up a SUNRISE demo storefront
   
   1. Go to [the demo storefront starter](https://github.com/commercetools/commercetools-sunrise-java-starter#deployment) and press the "Deploy to Heroku" Button. Heroku is not affiliated to commercetools but one of the simples way to get a web application running without operating full infrastructure. 
   2. In Heroku, fill "App Name" with what you want to be in the URL of the demo shop (better not leave blank)
   3. The Starter UI is configures with environment variables:
     * Fill Project Key, Client ID and Client Secret of your platform project (they are safe here if you have a secure Heroku password)
     * Just don't bother about the other values (you will change them later as suitable). You can find them as "config vars" in the app / dyno settings. 
   4. Press "deploy for free", wait a while and hooray, you should have a working, but empty online store live on the web.  
   
## Phase 4: Enter or Import Categories

> First of all: Think well about which scope of the catalog you want to edit products for. Empty categories are not impressive. 

> Attention: The SUNRISE webshop does only periodically refresh the category tree from the platform. To force refresh, restart the Heroku app, or (much better) call the `/categories/refresh` in the webshop (e.g. http://my-sales-demo.herokuapp.com/categories/refresh )   

Alternatives: 

### a) use the sample category tree
The [SUNRISE demo dataset](https://github.com/commercetools/commercetools-sunrise-data)  includes a standalone category import: ``npm run import:categories`` 

### b) enter manually
Use [Merchant Center](https://mc.commercetools.com/) 

### c) import automatically. 

Please follow the documentation of the [commercetools CLI](https://github.com/sphereio/sphere-node-cli). Typical Issues to consider: 

* Plan well, which identifier to use to map the products to the categories (externalId is usually the safest way)
* Make sure that your category feed is ordered so that the parent categories are there before their children.  If you category feed is randomly ordered, do a two-pass import by first importing all categories "flat" without parent association and then running it again with parent associations, effectively building the tree). 

## Phase 5: Model the product type

> Tip: consider to first import only the minimum required data before adding lots of attributes. 
> The SUNRISE demo works well without any custom attributes. 

> Tip: if you work with product variants, you need at least the attribute(s) that differentiate the variants so the Frontend can show the variant selector dropdowns. 
> This attribute should have the constraint "combination unique" set. 

You can interactively model the product type in the Merchant Center. 
 * Take your time to understand the Attribute types and Constraints (precise documentation can be found here: http://dev.commercetools.com/http-api-projects-productTypes.html#attribute-definition )
 * From you analysis (see above), rather choose just a few core attributes and do them right instead of trying completeness. E.g. a well configured "Enum" creates a dropdown in the PIM section of the Merchant Center, which is more impressive than an array of 50 text fields.
 * Type specific hints:
  * to fill the localizable types with multiple language values from CSV, append e.g. `.de` or `.en` to the column name. 
  * Enum is nice, Localizable Enum is nicer (you can "translate" / "rename" the values in the source data feed).
  * Although powerful, avoid Nested and Reference attributes in demo imports unless they play a key role in the project. They are not as easy to import and harder to explain (esp. Nested)

After you modeled your default product type, go into the "Settings" tab in the Heroku Backend, click "Reveal Config Vars" and set the following: 

 * `HARD_SELECTABLE_ATTRIBUTES` = put the attribute name that differentiates the variants (or a comma separated list)
 * `DISPLAYED_ATTRIBUTES` = put all custom attributes names as a comma separated list to let the webshop show them in a table.

## Phase 6: Enter or Import Products
  
### Manually

Products are added in the Products Section of the [Merchant Center](https://admin.sphere.io). 

Data required for the web shop demo:

 * Name (at least in english)
 * Tax category ("default")
 * A sku 
 * At least one price for the currency used (leave country, customer group, etc in default "All" state)
 * The variant selection attribute you defined in the product type
 * Products do not have to be put in their parent categories manually (but you can)
 
Data recommended to be filled for the demo

 * A description 
 * At least one Image (please fill in the dimensions if you are entering an external URL instead of uploading the image) 
 * You can remove the random number from the end of the slug (SEO tab) to get nicer URLs


> Don't forget to publish your products (there's a special button in the "overview" tab)! 
> Once published, they become visible immediately, otherwise you will waste a lot of "it's a feature, not a bug" time.  

### Transform and Import a Datafeed

> *If* you plan to manually clean / transform the data e.g. in Excel or LibreOffice (instead of using the commercetools tools), skip the "Local Installation" Part and [continue here](#running-the-import).  

#### Local Installation

First of all, you need a newer [Node.js](https://nodejs.org/en/download/) runtime on your local machine (there are tips on using a package manager like Homebrew or apt at the bottom of the page).

Secondly, clone this GitHub repository to your machine and use it as the starting point for your working directory. 
It contains the necessary setup, helper scripts and sample files. 
As long as you don't push back to the commercetools GitHub, you can commit and version locally as you like. 
You can also choose to fork it on GitHub, but keep in mind that sales prospects don't really like the information public. 

To install the typically needed commercetools commandline tools, run

```
npm install
```
Now you have a project-local installation of all the commandline tools, located in `./node_modules/.bin/`. The following commands are available:
```
# transform CSVs:
./node_modules/.bin/csv-mapper
# import / sync products from CSV:
./node_modules/.bin/product-csv-sync
# import / sync categories form CSV:
./node_modules/.bin/category-sync
# import / sync products from JSON:
./node_modules/.bin/sphere-import
# generic CLI:
./node_modules/.bin/sphere
```

Now add your API client key and secret to the `.sphere-project-credentials.json` file so the CLI has them available. 

#### Transform the Product Data to the right format. 

 * first formally clean the input CSV. This repository contains [a self-documenting `transform-csv.sh` shell script with the usually necessary steps](transform-csv.sh). 
 * use [the CSV-Mapper](https://github.com/sphereio/csv-mapper/) (or a spreadsheet if you don't want to be able to reproduce the results with updated data) to
  * bring the CSV into the [format required by the CSV sync](https://github.com/sphereio/sphere-node-product-csv-sync#csv-format) 
  * A lot of real-world examples can be found in the [CSV-Mapper Examples File](./csv-mapping-examples.yaml) in this project.  
  * A good way to start is the [csv-mapping.yaml](./csv-mapping.yaml) config template. 

 * OR bring the JSON into the [format required by the JSON import](https://github.com/sphereio/sphere-product-import/blob/master/readme/product-import.md#sample-inputs) with your preferred tooling or programming language. 
   But be warned: that is usually too much work for a quick demo setup. 
  

#### Running the Import

[IMPEX Web GUI](https://impex.sphere.io/commands/product-import) is a cloud hosted version of the commercetools CLI toolings for your convenience. All CLIs are supported, but not the csv-mapper. 

## Phase 7: Customize the Look and CSS of the demo storefront. 

To customize the demo storefront, you will make use of the fact that the SUNRISE storefront has been directly deployed from GitHub to Heroku. 
This effectively creates a fork of the project on a Heroku git server. The only inconvenience is that to conveniently access that git repository you have to 

 * install the [Heroku Toolbelt](https://toolbelt.heroku.com/) (essentially a Heroku CLI)
 * create a working directory on your local machine, `cd` there and run:

```
heroku git:clone -a myapp
```

Now you have the source code on your computer (in a a new subdirectory). You can open the files there in any editor or ideally in an IDE.

The easiest setup is not run the frontend locally at all but let Heroku do all the work for you.  
Every time you push changes to Heroku, it will directly rebuild and redeploy the application and you will directly see the build log, errors etc. in your local console. 

So after you did your changes you can run:

```
$ git add -A
$ git commit -am "some comment"
$ git push
```
(the first command is only necessary if you have added new files)
  
Alternatively, start the frontend server on your local computer (see the documentation in the SUNRISE project for that). 

### Customize the look

Typical tasks for a demo: Change the color scheme, mood images and logos, but don't touch the layout (yet). 

The recommended way (least amout of learning curve and software setup complexity) is to try to do all changes in CSS and Javascript. 
For this reason, SUNRISE already contains two empty files especially for demo tasks:
 * `./public/stylesheet/sunrise.css` (loaded in the HTML head)
 * `./public/javascripts/sunrise.js` (loaded at at the end of the page load process)

#### Advanced customizations

The SUNRISE Java project contains good documentation on how to customize the project a bit or a lot or completely like in a production project.
 * https://github.com/sphereio/commercetools-sunrise-java/blob/master/ONBOARDING.md
 * esp. from here on: https://github.com/sphereio/commercetools-sunrise-java/blob/master/ONBOARDING.md#basic-customization

If you want to fully customize the theme or work on SCSS level, you should rather fork the complete [SUNRISE theme](https://github.com/sphereio/commercetools-sunrise-theme). But that is typically out of scope for a demonstration taks. 

### Customize Logic?

If you need to do this, you should reconsider whether you're still in demo mode or have migrated to doing a real project. 
We do not recommend directly migrating a demo to a live project. It's not because the software is bad, but rather because it is a bad idea to not do thorough analysis on the data modeling and overall technical project architecture. 
Commercetools does offer trainings to build web storefronts on the foundation of the SUNRISE framework. 
