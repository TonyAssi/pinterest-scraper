# Pinterest Scraper
Scrape high-quality images from [Pinterest](https://www.pinterest.com/) with javascript.

![Pinterest-Logo wine](https://github.com/user-attachments/assets/e2b42cc9-448e-49fb-90c7-9f2b3fe54bf8)

---

1. Open [Pinterest](https://www.pinterest.com/) in Google Chrome.

2. Open Chrome developer tools and go to the console tab.

3. Copy and paste this code into the console

```js
const delay = ms => new Promise(res => setTimeout(res, ms));

async function smartAutoScroll(maxTries = 10) {
  let seen = new Set();
  let stableCount = 0;

  for (let i = 0; i < 100; i++) {
    window.scrollBy(0, window.innerHeight);
    document.dispatchEvent(new MouseEvent("mousemove")); // fake activity
    await delay(1500);

    const current = Array.from(document.querySelectorAll('img')).map(img => img.src);
    const currentSet = new Set(current);

    const newOnes = [...currentSet].filter(x => !seen.has(x));
    newOnes.forEach(x => seen.add(x));

    console.log(`Scroll ${i + 1}: +${newOnes.length} new images`);

    if (newOnes.length === 0) {
      stableCount += 1;
    } else {
      stableCount = 0;
    }

    if (stableCount >= maxTries) break;
  }

  return [...seen];
}

function convertToHighRes(urls) {
  return [...new Set(urls.map(url =>
    url.replace(/\/\d+x\d+\//, '/originals/').replace(/\/\d+x\//, '/originals/')
  ))];
}

async function checkValidUrls(urls, batchSize = 100) {
  const valid = [];
  for (let i = 0; i < urls.length; i += batchSize) {
    const batch = urls.slice(i, i + batchSize);
    console.log(`Checking batch ${i / batchSize + 1}`);
    const results = await Promise.all(batch.map(url =>
      fetch(url, { method: 'HEAD' })
        .then(res => res.ok ? url : null)
        .catch(() => null)
    ));
    valid.push(...results.filter(Boolean));
    await delay(500);
  }
  return valid;
}

function downloadCSV(urls) {
  const csvContent = "data:text/csv;charset=utf-8," + urls.map(u => `"${u}"`).join("\n");
  const encodedUri = encodeURI(csvContent);
  const link = document.createElement("a");
  link.setAttribute("href", encodedUri);
  link.setAttribute("download", "pinterest_highres.csv");
  document.body.appendChild(link);
  link.click();
}

(async () => {
  console.log("🌀 Scrolling smartly with interaction...");
  const lowRes = await smartAutoScroll();
  console.log(`📷 Total images collected: ${lowRes.length}`);
  const highRes = convertToHighRes(lowRes);
  console.log(`🔎 Checking high-res URLs (${highRes.length})...`);
  const valid = await checkValidUrls(highRes);
  console.log(`✅ ${valid.length} working high-res images. Downloading CSV...`);
  downloadCSV(valid);
})();

```

4. Allow some time for the script to scroll and collect images. After it's done it'll autodownload a .csv file.
