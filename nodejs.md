
# [puppeteer](https://github.com/GoogleChrome/puppeteer)
  chrome 기반 화면테스트
```c
const puppeteer = require('puppeteer');

(async () => {
  const browser = await puppeteer.launch();
  const page = await browser.newPage();
  await page.goto('https://example.com');
  await page.screenshot({path: 'example.png'});

  await browser.close();
})();
```
