# Firecrawl Node SDK (Fork)

This is a **fork** of the official Firecrawl Node SDK, created to provide additional support and customizations, such as the ability to add custom headers when initializing the SDK instance.

The fork was made from the original repository: [https://github.com/mendableai/firecrawl](https://github.com/mendableai/firecrawl).

### How the Fork Was Made

The fork was created by copying only the JavaScript SDK folder (`apps/js-sdk/firecrawl`) into this repository, with modifications to allow custom headers to be passed when initializing the `FirecrawlApp`. These changes enable users to include, for example, authentication tokens or other custom headers in the API requests made by the SDK.

This fork maintains most of the original structure but includes the necessary modifications to support sending additional headers such as the `Authorization` header with tokens and other custom parameters.

## Installation

To install the Firecrawl Node SDK, you can use npm:

```bash
npm install firecrawl-selfhosted-sdk
````

## Usage

1. Get an API key from [firecrawl.dev](https://firecrawl.dev)
2. Set the API key as an environment variable named `FIRECRAWL_API_KEY` or pass it as a parameter to the `FirecrawlApp` class.

Additionally, you can pass custom headers, such as `Authorization`, when initializing the `FirecrawlApp` instance. This allows you to customize the request headers used for the API calls.

### Example of Using the SDK

Here's an example of how to use the SDK with error handling:

```js
import FirecrawlApp, { CrawlParams, CrawlStatusResponse } from '@fmoraes87/firecrawl-selfhosted-sdk';

const app = new FirecrawlApp({ 
  apiKey: "fc-YOUR_API_KEY", //optional
  headers: { //custom headers
    'Authorization': `Bearer ${process.env.FIRECRAWL_AUTH_TOKEN}`,
    'Custom-Header': 'CustomValue'
  }
});

// Scrape a website
const scrapeResponse = await app.scrapeUrl('https://firecrawl.dev', {
  formats: ['markdown', 'html'],
});

if (scrapeResponse) {
  console.log(scrapeResponse)
}

// Crawl a website
const crawlResponse = await app.crawlUrl('https://firecrawl.dev', {
  limit: 100,
  scrapeOptions: {
    formats: ['markdown', 'html'],
  }
})

console.log(crawlResponse)
```

### Scraping a URL

To scrape a single URL with error handling, use the `scrapeUrl` method. It takes the URL as a parameter and returns the scraped data as a dictionary.

```js
const url = "https://example.com";
const scrapedData = await app.scrapeUrl(url);
```

### Crawling a Website

To crawl a website with error handling, use the `crawlUrl` method. It takes the starting URL and optional parameters as arguments. The `params` argument allows you to specify additional options for the crawl job, such as the maximum number of pages to crawl, allowed domains, and the output format.

```js
const crawlResponse = await app.crawlUrl('https://firecrawl.dev', {
  limit: 100,
  scrapeOptions: {
    formats: ['markdown', 'html'],
  }
})
```

### Asynchronous Crawl

To initiate an asynchronous crawl of a website, utilize the AsyncCrawlURL method. This method requires the starting URL and optional parameters as inputs. The params argument enables you to define various settings for the asynchronous crawl, such as the maximum number of pages to crawl, permitted domains, and the output format. Upon successful initiation, this method returns an ID, which is essential for subsequently checking the status of the crawl.

```js
const asyncCrawlResult = await app.asyncCrawlUrl('mendable.ai', { excludePaths: ['blog/*'], limit: 5});
```

### Checking Crawl Status

To check the status of a crawl job with error handling, use the `checkCrawlStatus` method. It takes the job ID as a parameter and returns the current status of the crawl job

```js
const status = await app.checkCrawlStatus(id);
```

### Extracting structured data from a URL

With LLM extraction, you can easily extract structured data from any URL. We support zod schema to make it easier for you too. Here is how you to use it:

```js
import FirecrawlApp from "@fmoraes87/firecrawl-selfhosted-sdk";
import { z } from "zod";

const app = new FirecrawlApp({
  apiKey: "fc-YOUR_API_KEY", //optional
});

// Define schema to extract contents into
const schema = z.object({
  top: z
    .array(
      z.object({
        title: z.string(),
        points: z.number(),
        by: z.string(),
        commentsURL: z.string(),
      })
    )
    .length(5)
    .describe("Top 5 stories on Hacker News"),
});

const scrapeResult = await app.scrapeUrl("https://firecrawl.dev", {
  extractorOptions: { extractionSchema: schema },
});

console.log(scrapeResult.data["llm_extraction"]);
```

### Map a Website

Use `map_url` to generate a list of URLs from a website. The `params` argument lets you customize the mapping process, including options to exclude subdomains or to utilize the sitemap.

```js
const mapResult = await app.mapUrl('https://example.com') as MapResponse;
console.log(mapResult)
```

### Crawl a website with WebSockets

To crawl a website with WebSockets, use the `crawlUrlAndWatch` method. It takes the starting URL and optional parameters as arguments. The `params` argument allows you to specify additional options for the crawl job, such as the maximum number of pages to crawl, allowed domains, and the output format.

```js
// Crawl a website with WebSockets:
const watch = await app.crawlUrlAndWatch('mendable.ai', { excludePaths: ['blog/*'], limit: 5});

watch.addEventListener("document", doc => {
 console.log("DOC", doc.detail);
});

watch.addEventListener("error", err => {
 console.error("ERR", err.detail.error);
});

watch.addEventListener("done", state => {
 console.log("DONE", state.detail.status);
});
```

### Batch scraping multiple URLs

To batch scrape multiple URLs with error handling, use the `batchScrapeUrls` method. It takes the starting URLs and optional parameters as arguments. The `params` argument allows you to specify additional options for the batch scrape job, such as the output formats.

```js
const batchScrapeResponse = await app.batchScrapeUrls(['https://firecrawl.dev', 'https://mendable.ai'], {
  formats: ['markdown', 'html'],
})
```

#### Asynchronous batch scrape

To initiate an asynchronous batch scrape, utilize the `asyncBatchScrapeUrls` method. This method requires the starting URLs and optional parameters as inputs. The params argument enables you to define various settings for the scrape, such as the output formats. Upon successful initiation, this method returns an ID, which is essential for subsequently checking the status of the batch scrape.

```js
const asyncBatchScrapeResult = await app.asyncBatchScrapeUrls(['https://firecrawl.dev', 'https://mendable.ai'], { formats: ['markdown', 'html'] });
```

#### Batch scrape with WebSockets

To use batch scrape with WebSockets, use the `batchScrapeUrlsAndWatch` method. It takes the starting URL and optional parameters as arguments. The `params` argument allows you to specify additional options for the batch scrape job, such as the output formats.

```js
// Batch scrape multiple URLs with WebSockets:
const watch = await app.batchScrapeUrlsAndWatch(['https://firecrawl.dev', 'https://mendable.ai'], { formats: ['markdown', 'html'] });

watch.addEventListener("document", doc => {
 console.log("DOC", doc.detail);
});

watch.addEventListener("error", err => {
 console.error("ERR", err.detail.error);
});

watch.addEventListener("done", state => {
 console.log("DONE", state.detail.status);
});
```

## Error Handling

The SDK handles errors returned by the Firecrawl API and raises appropriate exceptions. If an error occurs during a request, an exception will be raised with a descriptive error message. The examples above demonstrate how to handle these errors using `try/catch` blocks.

## License

The Firecrawl Node SDK is licensed under the MIT License. This means you are free to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the SDK, subject to the following conditions:

* The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

Please note that while this SDK is MIT licensed, it is part of a larger project which may be under different licensing terms. Always refer to the license information in the root directory of the main project for overall licensing details.