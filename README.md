# Web Scraping With Laravel

[![Promo](https://github.com/luminati-io/LinkedIn-Scraper/raw/main/Proxies%20and%20scrapers%20GitHub%20bonus%20banner.png)](https://brightdata.com/) 

This guide explains how to perform web scraping using Laravel:

- [Prerequisites](#prerequisites)
- [How to Build a Web Scraping API in Laravel](#how-to-build-a-web-scraping-api-in-laravel)
  - [Step 1: Set up a Laravel Project](#step-1-set-up-a-laravel-project)
  - [Step 2: Initialize Your Scraping API](#step-2-initialize-your-scraping-api)
  - [Step 3: Install the Scraping Libraries](#step-3-install-the-scraping-libraries)
  - [Step 4: Download the Target Page](#step-4-download-the-target-page)
  - [Step 5: Inspect the Page Content](#step-5-inspect-the-page-content)
  - [Step 6: Prepare for Web Scraping](#step-6-prepare-for-web-scraping)
  - [Step 7: Implement Data Scraping](#step-7-implement-data-scraping)
  - [Step 8: Return the Scraped Data](#step-8-return-the-scraped-data)
  - [Step 9: Put It All Together](#step-9-put-it-all-together)
- [Next Steps](#next-steps)

## Is It Possible to Perform Web Scraping in Laravel?

[Laravel](https://laravel.com/) is a powerful PHP framework with an elegant syntax, making it ideal for building APIs for web scraping. It supports various scraping libraries, simplifying data extraction. 

Laravel’s scalability, easy integration, and strong MVC architecture keep scraping logic well-organized, making it great for complex or large-scale projects. For more details, see our guide on [web scraping in PHP](https://brightdata.com/blog/how-tos/web-scraping-php).

## Best Laravel Web Scraping Libraries

Here are some top libraries for web scraping in Laravel:

- [**BrowserKit**](https://github.com/symfony/browser-kit) – A Symfony component that simulates a web browser API for interacting with static HTML documents. It works with `DomCrawler` for efficient navigation and scraping.
- [**HttpClient**](https://github.com/symfony/http-client) – A Symfony HTTP client that integrates seamlessly with `BrowserKit` for sending requests.
- [**Guzzle**](https://github.com/guzzle/guzzle) – A powerful HTTP client for making web requests and handling responses. Useful for retrieving HTML documents. Learn [how to set up a proxy in Guzzle](https://brightdata.com/blog/how-tos/proxy-with-guzzle).
- [**Panther**](https://github.com/symfony/panther) – A headless browser for scraping dynamic sites that require JavaScript rendering or interaction.
## Prerequisites

To follow this tutorial for web scraping in Laravel, you need to meet the following prerequisites:

- [PHP 8+](https://www.php.net/downloads.php)
- [Composer](https://getcomposer.org/download/)

An IDE to code in PHP is also recommended.

## How to Build a Web Scraping API in Laravel

This section walks you through creating a Laravel web scraping API using the [Quotes scraping sandbox site](https://quotes.toscrape.com/). The scraping endpoint will:

1. Select quote HTML elements from the page  
2. Extract data from them  
3. Return the scraped data in JSON format  

Here’s what the target site looks like:

![Quotes to scrape page](https://brightdata.com/wp-content/uploads/2022/11/Quotes-to-Scrape-page-gif.gif)

**Step 1: Set up a Laravel project**

Open the terminal and launch the Composer [`create-command`](https://getcomposer.org/doc/03-cli.md#create-project) command below to initialize your Laravel web scraping application:

```bash
composer create-project laravel/laravel laravel-scraper
```

The `lavaral-scraper` folder will now contain a blank Laravel project. Load it in your favorite PHP IDE.

This is the file structure of your current backend:

![file structure in the backend](https://brightdata.com/wp-content/uploads/2024/08/file-structure-in-the-backend.png)

**Step 2: Initialize Your Scraping API**

Launch the [Artisan command](https://laravel.com/docs/11.x/artisan) below in the project directory to add a new Laravel controller:

```bash
php artisan make:controller HelloWorldController
```

This will create the following `ScrapingController.php` file in the `/app/Http/Controllers` directory:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class ScrapingController extends Controller

{

//

}
```

In `ScrapingController` file, add the following `scrapeQuotes()` method:

```
public function scrapeQuotes(): JsonResponse

{

// scraping logic...

return response()->json('Hello, World!');

}
```

For now, the method returns a placeholder `'Hello, World!'` JSON message.

Add the following import:

```php
use Illuminate\Http\JsonResponse;
```

Associate the `scrapeQuotes()` method to a dedicated endpoint by adding the following lines to `routes/api.php`:

```php
use App\Http\Controllers\ScrapingController;

Route::get('/v1/scraping/scrape-quotes', [ScrapingController::class, 'scrapeQuotes']);
```

Let's verify that the Laravel scraping API works as expected. Since the Laravel APIs are available under the `/api` path, the complete API endpoint is `/api/v1/scraping/scrape-quotes`.

Launch your Laravel application:

```php
php artisan serve
```

Your server should now be listening locally on port `8000`.

Use cURL to make a `GET` request to the `/api/v1/scraping/scrape-quotes` endpoint:

```bash
curl -X GET 'http://localhost:8000/api/v1/scraping/scrape-quotes'
```

You should get the following response:

```
"Hello, World!"
```

**Step 3: Install the scraping libraries**

Before installing any packages, determine which Laravel web scraping libraries suit your needs. Open the target site, inspect it using Developer Tools, and check the **Network → Fetch/XHR** section:

![Accessing the 'Fetch XHR' section](https://brightdata.com/wp-content/uploads/2024/08/accessing-the-Fetch-XHR-section.png)

Since the site does not make [AJAX requests](https://developer.mozilla.org/en-US/docs/Web/Guide/AJAX), it is a static page with data embedded in the HTML. A [headless browser](https://brightdata.com/blog/web-data/best-headless-browsers) is unnecessary, as it would add overhead.

For efficient scraping, use Symfony’s [`BrowserKit`](https://symfony.com/doc/current/http_client.html) and [`HttpClient`](https://symfony.com/doc/current/http_client.html). Install them with:

```bash
composer require symfony/browser-kit symfony/http-client
```

**Step 4: Download the target page**

Import `BrowserKit` and `HttpClient` in `ScrapingController`:

```php
use Symfony\Component\BrowserKit\HttpBrowser;

use Symfony\Component\HttpClient\HttpClient;
```

In `scrapeQuotes()`, initialize a new `HttpBrowser` object:

```php
$browser = new HttpBrowser(HttpClient::create());
```

`HttpBrowser` allows you to make HTTP requests while mimicking browser behavior, including cookie and session handling. However, it does not execute requests in a real browser.

Use the `request()` method to perform an HTTP GET request to the target URL:

```php
$crawler = $browser->request('GET', 'https://quotes.toscrape.com/');
```

The result will be a [`Crawler`](https://github.com/symfony/symfony/blob/7.1/src/Symfony/Component/DomCrawler/Crawler.php) object, which automatically parses the HTML document returned by the server. This class also provides node selection and data extraction capabilities.

You can verify that the above logic works by extracting the HTML of the page from the crawler:

```
$html = $crawler->outerHtml();
```

For testing, make your API return this data.

Your `scrapeQuotes()` function will now look like this:

```php
public function scrapeQuotes(): JsonResponse

{

// initialize a browser-like HTTP client

$browser = new HttpBrowser(HttpClient::create());

// download and parse the HTML of the target page

$crawler = $browser->request('GET', 'https://quotes.toscrape.com/');

// get the page outer HTML and return it

$html = $crawler->outerHtml();

return response()->json($html);

}
```

Your API will now return:

```html
<!DOCTYPE html>

<html lang="en">

<head>

<meta charset="UTF-8">

<title>Quotes to Scrape</title>

<link rel="stylesheet" href="/static/bootstrap.min.css">

<link rel="stylesheet" href="/static/main.css">

</head>

<!-- omitted for brevity ... -->
```

**Step 5: Inspect the page content**

To define the extraction logic, inspect the HTML structure of the target page.

1. Open [Quotes To Scrape](https://quotes.toscrape.com/).  
2. Right-click a quote element and select **Inspect** in DevTools.  
3. Expand the HTML and examine its structure:

![Inspecting the quote elements](https://brightdata.com/wp-content/uploads/2024/08/Inspecting-the-quote-elements-1024x814.png)

Each `.quote` element contains:  
- A `.text` node for the quote text  
- An `.author` node for the author’s name  
- Multiple `.tag` nodes for associated tags  

With these [CSS selectors](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Selectors), you can now extract the desired data in Laravel.

**Step 6: Get ready to perform web scraping**

Create a data structure where to store the scraped data. Use an array for that:

```php
quotes = []
```

Now use the `filter()` method from the `Crawler` class to select all quote elements:

```php
$quote_html_elements = $crawler->filter('.quote');
```

This returns all DOM nodes on the page that match the specified `.quote` CSS selector.

Next, iterate over them and get ready to apply the data extraction logic on each of them:

```php
foreach ($quote_html_elements as $quote_html_element) {

// create a new quote crawler

$quote_crawler = new Crawler($quote_html_element);

// scraping logic...

}
```

The `DOMNode` objects returned by `filter()` lack node selection methods. To work around this, create a local `Crawler` instance scoped to the specific HTML quote element.

For the code to function correctly, add the following import:

```php
use Symfony\Component\DomCrawler\Crawler;
```

**Step 7: Implement data scraping**

Inside the `foreach` loop:

1. Extract the data of interest from the `.text`, `.author`, and `.tag` elements
2. Populate a new `$quote` object with them
3. Add the new `$quote` object to `$quotes`

First, select the .text element inside the HTML quote element. Then, use the `text()` method to extract the inner text from it:

```php
$text_html_element = $quote_crawler->filter('.text');

$raw_text = $text_html_element->text();
```

Each quote is enclosed by the `\u201c` and `\u201d` special characters. You can remove them using the [`str_replace()`](https://www.php.net/manual/en/function.str-replace.php) PHP function as follows:

```php
$text = str_replace(["\u{201c}", "\u{201d}"], '', $raw_text);
```

Similarly, scrape the author info with this code:

```php
$author_html_element = $quote_crawler->filter('.author');

$author = $author_html_element->text();
```

Scraping the tags can be challenging. Since a single quote can have multiple tags, you need to define an array and scrape each tag individually:

```php
$tag_html_elements = $quote_crawler->filter('.tag');

$tags = [];

foreach ($tag_html_elements as $tag_html_element) {

$tag = $tag_html_element->textContent;

$tags[] = $tag;

}
```

Note that the `DOMNode` elements returned by `filter()` do not expose the `text()` method. Equivalently, they provide the `textContent` attribute.

Here is the entire Laravel data scraping logic:

```php
// create a new quote crawler

$quote_crawler = new Crawler($quote_html_element);

// perform the data extraction logic

$text_html_element = $quote_crawler->filter('.text');

$raw_text = $text_html_element->text();

// remove special characters from the raw text information

$text = str_replace(["\u{201c}", "\u{201d}"], '', $raw_text);

$author_html_element = $quote_crawler->filter('.author');

$author = $author_html_element->text();

$tag_html_elements = $quote_crawler->filter('.tag');

$tags = [];

foreach ($tag_html_elements as $tag_html_element) {

$tag = $tag_html_element->textContent;

$tags[] = $tag;

}
```

**Step 8: Return the scraped data**

Create a `$quote` object with the scraped data and add it to `$quotes`:

```php
$quote = [

'text' => $text,

'author' => $author,

'tags' => $tags

];

$quotes[] = $quote;
```

Next, update the API response data with the `$quotes` list:

```php
return response()->json(['quotes' => $quotes]);
```

At the end of the scraping loop, `$quotes` will contain:

```php
array(10) {

[0]=>

array(3) {

["text"]=>

string(113) "The world as we have created it is a process of our thinking. It cannot be changed without changing our thinking."

["author"]=>

string(15) "Albert Einstein"

["tags"]=>

array(4) {

[0]=>

string(6) "change"

[1]=>

string(13) "deep-thoughts"

[2]=>

string(8) "thinking"

[3]=>

string(5) "world"

}

}

// omitted for brevity...

[9]=>

array(3) {

["text"]=>

string(48) "A day without sunshine is like, you know, night."

["author"]=>

string(12) "Steve Martin"

["tags"]=>

array(3) {

[0]=>

string(5) "humor"

[1]=>

string(7) "obvious"

[2]=>

string(6) "simile"

}

}

}
```

This data will then be serialized into JSON and returned by the Laravel scraping API.

**Step 9: Put it all together**

Here is the final code of the `ScrapingController` file in Laravel:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

use Illuminate\Http\JsonResponse;

use Symfony\Component\BrowserKit\HttpBrowser;

use Symfony\Component\HttpClient\HttpClient;

use Symfony\Component\DomCrawler\Crawler;

class ScrapingController extends Controller

{

public function scrapeQuotes(): JsonResponse

{

// initialize a browser-like HTTP client

$browser = new HttpBrowser(HttpClient::create());

// download and parse the HTML of the target page

$crawler = $browser->request('GET', 'https://quotes.toscrape.com/');

// where to store the scraped data

$quotes = [];

// select all quote HTML elements on the page

$quote_html_elements = $crawler->filter('.quote');

// iterate over each quote HTML element and apply

// the scraping logic

foreach ($quote_html_elements as $quote_html_element) {

// create a new quote crawler

$quote_crawler = new Crawler($quote_html_element);

// perform the data extraction logic

$text_html_element = $quote_crawler->filter('.text');

$raw_text = $text_html_element->text();

// remove special characters from the raw text information

$text = str_replace(["\u{201c}", "\u{201d}"], '', $raw_text);

$author_html_element = $quote_crawler->filter('.author');

$author = $author_html_element->text();

$tag_html_elements = $quote_crawler->filter('.tag');

$tags = [];

foreach ($tag_html_elements as $tag_html_element) {

$tag = $tag_html_element->textContent;

$tags[] = $tag;

}

// create a new quote object

// with the scraped data

$quote = [

'text' => $text,

'author' => $author,

'tags' => $tags

];

// add the quote object to the quotes array

$quotes[] = $quote;

}

var_dump($quotes);

return response()->json(['quotes' => $quotes]);

}

}
```

Let's test it. Start your Laravel server:

```bash
php artisan serve
```

Make a GET request to the `/api/v1/scraping/scrape-quotes` endpoint:

```bash
curl -X GET 'http://localhost:8000/api/v1/scraping/scrape-quotes'
```

You will get the following result:

```
{

"quotes": [

{

"text": "The world as we have created it is a process of our thinking. It cannot be changed without changing our thinking.",

"author": "Albert Einstein",

"tags": [

"change",

"deep-thoughts",

"thinking",

"world"

]

},

// omitted for brevity...

{

"text": "A day without sunshine is like, you know, night.",

"author": "Steve Martin",

"tags": [

"humor",

"obvious",

"simile"

]

}

]

}
```

## Next steps

This API is a basic example of Laravel's web scraping capabilities. To improve and scale your project, consider these enhancements:

- **Implement web crawling** – The target site spans multiple pages. Use web crawling to retrieve all quotes efficiently.  
- **Schedule scraping tasks** – Automate data collection by scheduling API calls, storing data in a database, and keeping it up to date.  
- **Integrate proxies** – Avoid IP bans by using residential proxies to distribute requests and bypass anti-scraping measures.  

## Ethical Web Scraping

Web scraping is a powerful tool for data collection, but it must be done ethically and responsibly. Follow these best practices to ensure compliance and avoid harming target sites:

- **Review the site’s terms of service** – Check for guidelines on data usage, copyright, and intellectual property before scraping.  
- **Respect `robots.txt` rules** – Follow the site's crawling instructions to maintain ethical scraping practices.  
- **Scrape only public data** – Avoid restricted content requiring authentication, as scraping private data may have legal consequences.  
- **Limit request frequency** – Prevent server overload and rate limiting by pacing requests and adding random delays.  
- **Use reputable scraping tools** – Choose well-maintained tools that follow ethical scraping guidelines.

## Conclusion

Web scraping with Laravel is simple and takes only a few lines of code. However, most sites protect their data with anti-bot and anti-scraping solutions. To work around that, you can use [**Web Unlocker**](https://brightdata.com/products/web-unlocker), out unlocking API that can seamlessly return the clean HTML of any page, circumventing any anti-scraping measures.

Sign up now and start your free trial.
