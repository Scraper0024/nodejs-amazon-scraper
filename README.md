# nodejs-amazon-scraper
Learn how to build a powerful JavaScript web scraper with Node.js in 2025. This step-by-step tutorial covers setup, handling dynamic content, avoiding detection, and optimizing performance. Perfect for web scraping beginners and experts!

Amazon is a leading global e-commerce platform with valuable product data for market research and price monitoring. Using JavaScript and Node.js with Playwright, we can build a JavaScript web scraper to extract this data, but challenges like JavaScript rendering, anti-bot detection, and CAPTCHAs make manual scraping difficult.

Scrapeless's Amazon Scraping API offers a faster, more reliable solution by handling CAPTCHAs, proxies, and website unlocking, ensuring real-time, accurate data without detection. This tutorial will cover both approaches, showing why Scrapeless is the best choice for efficient Amazon data scraping. 🚀


## Challenges in Scraping Amazon
Scraping Amazon comes with several challenges due to its strong anti-bot protections:
- **JavaScript Rendering**: Amazon heavily relies on JavaScript, making it difficult to extract data with simple HTTP requests. A JavaScript scraper using Playwright or similar tools is required to render dynamic content.
- **CAPTCHA Protection**: Frequent CAPTCHA challenges interrupt scraping and require solving mechanisms to continue data extraction.
- **IP Blocking & Rate Limits**: Amazon detects and blocks repeated requests from the same IP, necessitating rotating proxies or other evasion techniques.
- **Frequent Website Changes**: Amazon updates its website structure regularly, which can break scraping scripts and require constant maintenance.

## How to Build Your JavaScript scraper with Node.js?
In the following example, we will use JavaScript to scrape Amazon and store the extracted data in a local JSON file.

### Preparing for Scraping
We need to set up the necessary Node.js web scraper tools at the very beginning:

In this article, I will use `Playwright`, a highly active open-source project with numerous contributors. Developed by Microsoft, it supports multiple browsers (Chromium, Firefox, and WebKit) and multiple programming languages (Node.js, Python, .NET, and Java), making it one of the most popular JavaScript scraping frameworks today.

Here, make sure your Node.js version is 18 or higher, then run the following script to install Playwright:

```Bash
# pnpm
pnpm create playwright
# yarn
yarn create playwright
# npm
npm init playwright@latest
```

> Frustrated on web blocking and headache project building?
Join [**Our Community**](https://discord.gg/8ajWRhtGUj) and get the effective solution with **Free Trial**!

### Step 1. Inspect the Target Page
Before scraping, try visiting **https://www.amazon.com/**. If this is your first time accessing the site, you may encounter a CAPTCHA.

![visit amazon.com](https://assets.scrapeless.com/prod/posts/nodejs-amazon-scraper/1af14cb79e9fecc749007d381c70725f.png)

But don’t worry—we don’t need to go through the hassle of finding a CAPTCHA-solving tool. Instead, simply access the Amazon domain specific to your region or your proxy’s location, and you won’t trigger a CAPTCHA.

For example, let’s visit https://www.amazon.co.uk/, the UK Amazon domain. You’ll see the page loads smoothly. Now, try entering your desired product keyword in the search bar at the top or directly access the search results via a URL, such as:

```Bash
https://www.amazon.co.uk/s?k=jacket
```

In the URL, the value after `/s?k=` represents the product keyword. By accessing the above URL, you’ll see shirt-related products on Amazon.

Now, open Developer Tools (F12) to inspect the HTML structure of the page. Use the cursor to highlight elements and identify the data we’ll need to scrape later.

![inspect the HTML structure](https://assets.scrapeless.com/prod/posts/nodejs-amazon-scraper/416bac69906857091308aa55d14f9e11.png)

### Step 2. Writing the Script
First, let's add an initial piece of code at the top of the script. The following code takes the first script argument as the Amazon product keyword, which will be used for scraping later:

```JavaScript
const productName = process.argv.slice(2);

if (productName.length !== 1) {console.error('product name CLI arguments missing!');
  process.exit(2);
}
```

Next, we need to:
- Import Playwright to interact with the browser.
- Navigate to the Amazon search results page.
- Add a screenshot to verify successful access.

```JavaScript
import playwright from "playwright";

const browser = await playwright.chromium.launch()

const page = await browser.newPage();

await page.goto(`https://www.amazon.co.uk/s?k=${productName}`);

// Add a screenshot for debugging
await page.screenshot({ path: 'amazon_page.png' })
```

Now, we'll use `page.$$` to fetch all product containers, loop through them, and extract the relevant data. This data is then stored in the `productDataList` array and printed:

```JavaScript
// Get all search result containers
const productContainers = await page.$$('div[data-component-type="s-search-result"]')

const productDataList = []

// Extract product details: title, rating, image URL, and price
for (const product of productContainers) {

  async function safeEval(selector, evalFn) {
    try {
      return await product.$eval(selector, evalFn);
    } catch (e) {
      return null;
    }
  }

  const title = await safeEval('.s-title-instructions-style > h2 > a > span', node => node.textContent)
  const rate = await safeEval('a > i.a-icon.a-icon-star-small > span', node => node.textContent)
  const img = await safeEval('span[data-component-type="s-product-image"] img', node => node.getAttribute('src'))
  const price = await safeEval('div[data-cy="price-recipe"] .a-offscreen', node => node.textContent)

  productDataList.push({ title, rate, img, price })
}

console.log('amazon_product_data_list :', productDataList);

await browser.close();
```

Run the script using:
```Bash
node amazon.js jacket
```

If successful, the console will print the extracted product data.

![scraping result](https://assets.scrapeless.com/prod/posts/nodejs-amazon-scraper/09a88872ad404d57056e7c75d50ca215.png)

### Step 3. Saving Scraped Data as a JSON File
Simply printing data to the console isn't enough for proper analysis. Let's save the extracted data into a JSON file using Node.js's `fs` module:

```JavaScript
import fs from 'fs'

function saveObjectToJson(obj, filename) {
  const jsonString = JSON.stringify(obj, null, 2)
  fs.writeFile(filename, jsonString, 'utf8', (err) => {
    err ? console.error(err) : console.log(`File saved successfully: ${filename}`);
  });
}

saveObjectToJson(productDataList, 'amazon_product_data.json')
```

Well, let's figure out the full script:

```JavaScript
import playwright from "playwright";
import fs from 'fs'

const productName = process.argv.slice(2);

if (productName.length !== 1) {
  console.error('product name CLI arguments missing!');
  process.exit(2);
}

const browser = await playwright.chromium.launch()

const page = await browser.newPage();

await page.goto(`https://www.amazon.co.uk/s?k=${productName}`);

// Add a screenshot for debugging
await page.screenshot({ path: 'amazon_page.png' })

// Get all search result containers
const productContainers = await page.$$('div[data-component-type="s-search-result"]')

const productDataList = []

// Extract product details: title, rating, image URL, and price
for (const product of productContainers) {

  async function safeEval(selector, evalFn) {
    try {
      return await product.$eval(selector, evalFn);
    } catch (e) {
      return null;
    }
  }

  const title = await safeEval('.s-title-instructions-style  > a > h2 > span', node => node.textContent)
  const rate = await safeEval('a > i.a-icon.a-icon-star-small > span', node => node.textContent)
  const img = await safeEval('span[data-component-type="s-product-image"] img', node => node.getAttribute('src'))
  const price = await safeEval('div[data-cy="price-recipe"] .a-offscreen', node => node.textContent)

  productDataList.push({ title, rate, img, price })
}

console.log('amazon_product_data_list :', productDataList);

function saveObjectToJson(obj, filename) {
  const jsonString = JSON.stringify(obj, null, 2)
  fs.writeFile(filename, jsonString, 'utf8', (err) => {
    err ? console.error(err) : console.log(`File saved successfully: ${filename}`);
  });
}

saveObjectToJson(productDataList, 'amazon_product_data.json')

await browser.close();
```

Now, after running the script, **the data will not only be printed on the console but also saved as a JSON file** (`amazon_product_data.json`).

![Save in a JSON file](https://assets.scrapeless.com/prod/posts/nodejs-amazon-scraper/641d9b7d3d351b07d84d3f8a8e39ada7.png)

## Avoid Getting Blocked While scraping Amazon
Scraping data from Amazon can be challenging due to its strict anti-bot measures, but using [Scrapeless's Web Unlocker](https://www.scrapeless.com/en/product/web-unlocker) helps to effectively bypass these restrictions.

Amazon employs bot detection techniques such as IP rate limiting, browser fingerprinting, and CAPTCHA verification to prevent automated access. Scrapeless's Web Unlocker overcomes these obstacles by rotating residential proxies, mimicking real user behavior, and handling dynamic content rendering.

By integrating Scrapeless with headless browsers such as playwright or Playwright, users can scrape Amazon's product data without being blocked, ensuring a seamless and efficient data extraction process.

## JavaScript scraping Best Practices and Considerations

When building a web scraper in JavaScript, it's crucial to optimize efficiency, handle dynamic content, and avoid common pitfalls. Here are some key best practices and considerations:
1. **Handling JavaScript-Rendered Pages**
Many modern websites load content dynamically using JavaScript. Traditional HTTP requests (like Axios or Fetch) won't capture such content. Instead, use headless browsers like playwright, Playwright, or Selenium to render and extract data from JavaScript-heavy pages.
2. **Managing Concurrency for Faster scraping**
Running scrapers sequentially can be slow. Implement concurrency by launching multiple parallel scrape tasks to improve performance. Use async/await with Promises and manage a queue system to balance the scrape load efficiently.
3. **Respecting Robots.txt and Website Policies**
Before scraping, check a website's `robots.txt` file to determine its scraping rules. Ignoring these rules may result in IP bans or legal issues. Additionally, consider using **user-agent rotation and request throttling** to minimize the impact on the target server.
4. **Avoiding Web Application Firewalls (WAFs)**
Websites deploy WAFs like Akamai, Cloudflare, and PerimeterX to block automated traffic. Techniques like **session persistence, browser fingerprinting evasion, and CAPTCHA-solving tools** can help mitigate detection and blocking.
5. **Efficient URL Management and Deduplication**
Ensure your scraper does not revisit the same URLs multiple times by maintaining a visited URLs set. Implement canonicalization techniques to normalize URLs and prevent duplicate data collection.
6. **Handling Pagination and Infinite Scrolling**
Websites often use pagination or infinite scrolling to load content dynamically. Identify the pagination structure (e.g., `?page=2`) or use headless browsers to simulate scrolling and extract all content.
7. **Data Extraction and Storage Optimization**
After extracting data, format it properly and store it efficiently. Save structured data in JSON, CSV, or databases like MongoDB or PostgreSQL for better processing and analysis.
8. **Distributed scraping for Large-Scale Scraping**
For large-scale scraping tasks, distribute the workload across multiple machines or cloud instances using **queue-based scraping frameworks or cloud browser solutions**. This prevents overloading a single system and improves fault tolerance.

## Using Scrapeless Scraping API for a Faster, More Reliable Solution
By integrating Scrapeless with headless browsers such as Playwright, users can build a scraper in JavaScript to extract Amazon product data without being blocked, ensuring a seamless and efficient data extraction process.
### Features:
✅ Get instant access to the latest data with just one API call.
✅ Over 200 concurrent requests per second with more than 100M requests per month.
✅ Each request takes an average of 5 seconds, ensuring real-time data retrieval without caching.
✅ Supports customized scraping rules to meet different needs
✅ Supports multi-platform scraping, in addition to Amazon, it can also support other e-commerce platforms: [Shopee](https://www.scrapeless.com/en/blog/how-to-scrape-product-data-from-shopee), [Shein](https://www.scrapeless.com/en/blog/how-to-scrape-shein-data), etc.
✅ Pay only for successful searches

### Why choose Scrapeless?
- **Real-Time Data**: Ensures up-to-date and accurate product listings.
- **Integrated Website Unlocking**: Automatically bypasses restrictions and CAPTCHAs.
- **Legality**: Scrapeless provides a legal and compliant way to access search results.
- **Reliability**: The API uses sophisticated techniques to avoid detection, ensuring uninterrupted data collection.
- **Ease of Use**: Scrapeless offers a simple API that integrates easily with Python, making it ideal for developers who need quick access to search result data.
- **Customizable**: You can tailor the results to your needs, such as specifying the type of content (e.g., organic listings, ads, etc.).

### Is Scrapeless Scraping API costly?
Scrapeless offers a reliable and scalable web scraping platform at [competitive prices](https://www.scrapeless.com/en/pricing) ( vs. [Zenrows](https://www.zenrows.com/pricing) & [Apify](https://apify.com/pricing)), ensuring excellent value for its users:
- **Scraping Browser**: From $0.09 per hour
- **Scraping API**: From $0.8 per 1k URLs
- **Web Unlocker**: $0.20 per 1k URLs
- **Captcha Solver**: From $0.80 per 1k URLs
- **Proxies**: $2.80 per GB

By [**subscribing**](https://app.scrapeless.com/passport/login?utm_source=official&utm_medium=blog&utm_campaign=nodejs-amazon-scraper), you can enjoy discounts of up to **20% discount** on each service.

### How to implement Scrapeless's Amazon Scraping API
If you are looking to extract ASIN numbers from Amazon product pages, Scrapeless provides a simple and effective way to do so. By using Scrapeless's Amazon Scraper API, you can easily get ASIN numbers along with other key product details.

**Step 1**. Log in to [**Scrapeless**](https://app.scrapeless.com/passport/login?utm_source=official&utm_medium=blog&utm_campaign=nodejs-amazon-scraper).

**Step 2**. Click Scraping API > **select Amazon** to enter the Shopee scraping page.

![Amazon Scraping API](https://assets.scrapeless.com/prod/posts/nodejs-amazon-scraper/8c5d4c8074cf4f983a9e1c1e2e5f1638.png)

**Step 3**. Paste the link to the **Amazon product page** you want to crawl into the input box. And select the **type of data** to crawl.

![configuration and scraping](https://assets.scrapeless.com/prod/posts/nodejs-amazon-scraper/b4a437d761317aefdf8216d833c9f06b.png)

**On the tool page, you can select the type of data to crawl:**
- **Seller**: Crawl seller information, including seller name, rating, contact information, etc.
- **Product**: Crawl product details such as title, price, rating, comments, etc.
- **Keywords**: Crawl keywords related to the product to help you analyze the product's SEO and market trends.

**Step 4**. After confirming that the input link and selected data type are correct, click the "**Start Scraping**" button. The system will start crawling data and display the crawled results on the panel on the right side of the page.

![Start Scraping](https://assets.scrapeless.com/prod/posts/nodejs-amazon-scraper/fd847c265a4c509f985d0dd595082515.png)

**Step 5**. After the crawling is completed, you can view the crawled data in the panel on the right. The results will be displayed in a clear format for easy analysis.

![scraping result](https://assets.scrapeless.com/prod/posts/nodejs-amazon-scraper/9a8cd03068319c28b72d2a434c95354c.png)

If you need to crawl other products, click Continue to enter a new Amazon link and repeat the above steps!

**You can also directly [integrate our codes](https://apidocs.scrapeless.com/api-12554367) into your project**:
> **Node.js**

```JavaScript
const https = require('https');

class Payload {
    constructor(actor, input) {
        this.actor = actor;
        this.input = input;
    }
}

function sendRequest() {
    const host = "api.scrapeless.com";
    const url = `https://${host}/api/v1/scraper/request`;
    const token = " "; // API Token

    const inputData = {
        action: "product",
        url: " " // Product URL
    };

    const payload = new Payload("scraper.amazon", inputData);
    const jsonPayload = JSON.stringify(payload);

    const options = {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json',
            'x-api-token': token
        }
    };

    const req = https.request(url, options, (res) => {
        let body = '';

        res.on('data', (chunk) => {
            body += chunk;
        });

        res.on('end', () => {
            console.log("body", body);
        });
    });

    req.on('error', (error) => {
        console.error('Error:', error);
    });

    req.write(jsonPayload);
    req.end();
}

sendRequest();
```

> **Python**

```Python
import json
import requests

class Payload:
    def __init__(self, actor, input_data):
        self.actor = actor
        self.input = input_data

def send_request():
    host = "api.scrapeless.com"
    url = f"https://{host}/api/v1/scraper/request"
    token = " " ## API Token

    headers = {
        "x-api-token": token
    }

    input_data = {
        "action": "product",
        "url": " " ## Product URL
    }

    payload = Payload("scraper.amazon", input_data)

    json_payload = json.dumps(payload.__dict__)

    response = requests.post(url, headers=headers, data=json_payload)

    if response.status_code != 200:
        print("Error:", response.status_code, response.text)
        return

    print("body", response.text)

if __name__ == "__main__":
    send_request()
```

> **Golang**
```Go
package main

import (
        "bytes"
        "encoding/json"
        "fmt"
        "io/ioutil"
        "net/http"
)

type Payload struct {
        Actor string         `json:"actor"`
        Input map[string]any `json:"input"`
}

func sendRequest() error {
        host := "api.scrapeless.com"
        url := fmt.Sprintf("https://%s/api/v1/scraper/request", host)
        token := " " // API Token
        
        headers := map[string]string{"x-api-token": token}
        
        inputData := map[string]any{
                "action": "product",
                "url":    " ", // Product URL
        }
        
        payload := Payload{
                Actor: "scraper.amazon",
                Input: inputData,
        }
        
        jsonPayload, err := json.Marshal(payload)
        if err != nil {
                return err
        }
        
        req, err := http.NewRequest("POST", url, bytes.NewBuffer(jsonPayload))
        if err != nil {
                return err
        }
        
        for key, value := range headers {
                req.Header.Set(key, value)
        }
        
        client := &http.Client{}
        resp, err := client.Do(req)
        if err != nil {
                return err
        }
        defer resp.Body.Close()
        
        body, err := ioutil.ReadAll(resp.Body)
        if err != nil {
                return err
        }
        fmt.Printf("body %s\n", string(body))
        
        return nil
}

func main() {
        err := sendRequest()
        if err != nil {
                fmt.Println("Error:", err)
                return
        }
}
```

## Conclusion: The Best Way to Scrape Amazon in 2025
Building a JavaScript scraper with Node.js gives full control over the scraping process but comes with challenges like handling dynamic content, solving CAPTCHAs, managing proxies, and avoiding detection. It requires significant technical effort and ongoing maintenance.

For large-scale, efficient, and undetectable scraping, Scrapeless's API is the best choice. It eliminates the complexities of bot detection, making data extraction effortless and scalable!

🚀 [**Try Scrapeless's Amazon Scraping API today**](https://app.scrapeless.com/passport/login?utm_source=official&utm_medium=blog&utm_campaign=nodejs-amazon-scraper) for a fast, reliable, and worry-free scraping experience!
