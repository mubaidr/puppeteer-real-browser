# puppeteer-real-browser

This package prevents Puppeteer from being detected as a bot in services like Cloudflare and allows you to pass captchas without any problems. It behaves like a real browser.

## Warnings

1. If you want to change the fingerprint values of the browser, please check this library: https://github.com/pavlealeksic/puppeteer-afp
   You can check whether it has changed or not from this link https://fingerprint.com/demo/.

2. It serves on a port on localhost to act like a real browser. The port must be closed when the process is finished. To close it, simply call browser.close(). In this method, operations to close the port are performed.

3. If you want the cloudflare Captcha to be skipped automatically, you can use this code. [CODE](https://github.com/zfcsoftware/youtube_lessons_resources/blob/main/puppeteer_cloudflare_bypass/index.js)

4. This library is stable when using headless false in windows and headless true in linux.

## installation

```bash
npm i puppeteer-real-browser
```

If it will run on linux you will also need to install xvfb.

```bash
sudo apt-get install xvfb
```

## Include

### commanjs

```js
const start = async () => {
  var { puppeteerRealBrowser } = await import('puppeteer-real-browser')
  const { page, browser } = await puppeteerRealBrowser({})
}
```

### Module

```js
import { puppeteerRealBrowser } from 'puppeteer-real-browser'

const { page, browser } = await puppeteerRealBrowser({})
```

## Usage

This package has 2 types of use. The first one opens the browser and connects with puppeteer. In this usage you cannot install chrome plugin or set puppeteer launch settings. If you don't need these, the first use is the best and lightest. Use 2 runs chromium. Then it opens a new browser with puppeteer.launch and connects to chromium. In total you have 2 browsers open. But you can use the same commands as with puppeteer.launch. Use 2 consumes more resources.

### Default Usage

```js
import { puppeteerRealBrowser } from 'puppeteer-real-browser'

// You should use it if you want the fingerprint values of the page to be changed.
// import puppeteerAfp from 'puppeteer-afp'

puppeteerRealBrowser({
  headless: false, // (optional) The default is false. If true is sent, the browser opens incognito. If false is sent, the browser opens visible.

  action: 'default', // (optional) If default, it connects with puppeteer by opening the browser and returns you the page and browser. if socket is sent, it returns you the browser url to connect to.

  executablePath: 'default', // (optional) If you want to use a different browser instead of Chromium, you can pass the browser path with this variable.

  // (optional) If you are using a proxy, you can send it as follows.

  // proxy:{
  //     host:'<proxy-host>',
  //     port:'<proxy-port>',
  //     username:'<proxy-username>',
  //     password:'<proxy-password>'
  // }

  args: [], // (optional) The default is []. Arguments to pass to underlying browser instance
})
  .then(async (response) => {
    const { browser, page } = response

    // You should use it if you want the fingerprint values of the page to be changed.
    // puppeteerAfp(page);

    await page.goto('<url>')
  })
  .catch((error) => {
    console.log(error.message)
  })
```

### Manageable Usage

This method is not recommended. Open a hidden chromium and connect to it with puppeteer. It consumes more resources. You can use it when you need to do things like installing a Chrome extension. You should use it by default unless you have to.

```bash
npm i puppeteer-real-browser puppeteer-extra-plugin-stealth
```

```js
import { puppeteerRealBrowser } from 'puppeteer-real-browser'
import puppeteer from 'puppeteer-extra'
import StealthPlugin from 'puppeteer-extra-plugin-stealth'
puppeteer.use(StealthPlugin())

puppeteerRealBrowser({
  headless: true, // (optional) The default is false. If true is sent, the browser opens incognito. If false is sent, the browser opens visible.

  action: 'socket', // (optional) If default, it connects with puppeteer by opening the browser and returns you the page and browser. if socket is sent, it returns you the browser url to connect to.

  executablePath: 'default', // (optional) If you want to use a different browser instead of Chromium, you can pass the browser path with this variable.

  // (optional) If you are using a proxy, you can send it as follows.

  // proxy:{
  //     host:'<proxy-host>',
  //     port:'<proxy-port>',
  //     username:'<proxy-username>',
  //     password:'<proxy-password>'
  // }
})
  .then(async (response) => {
    const { browserWSEndpoint, userAgent, closeSession, chromePath } = response

    const browser = await puppeteer.launch({
      targetFilter: (target) => !!target.url(),
      browserWSEndpoint: browserWSEndpoint,
      ignoreHTTPSErrors: true,
      headless: false,
      args: ['--no-sandbox', '--start-maximized', '--window-size=1920,1040'],
      // ... puppeteer args
    })

    const pages = await browser.pages()
    const page = pages[0]

    // If you cannot pass waf in this way, you can try the following user agents respectively.

    // await page.setUserAgent(userAgent);
    // await page.setUserAgent(undefined);

    await page.setViewport({
      width: 1920,
      height: 1080,
    })

    // You can use it if you need to log in to a proxy.
    // await page.authenticate({ username: proxy.username, password: proxy.password });

    await page.goto('<url>')
    // await closeSession()
    // await browser.close()
  })
  .catch((error) => {
    console.log(error.message)
  })
```

# Disclaimer of Liability

This library was created to understand how scanners like puppeteer are detected and to teach how to prevent detection. Its purpose is purely educational. Illegal use of the library is prohibited. The user is responsible for any problems that may arise. The repo owner accepts no responsibility.

# Thank You

[@JimmyLaurent](https://github.com/JimmyLaurent)
Inspired by his cloudflare-scraper library.
