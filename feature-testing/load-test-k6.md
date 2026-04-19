```
import http from "k6/http";
import { sleep, check } from "k6";

export const options = {
  stages: [
    { duration: "30s", target: 500 }, // warm up
    { duration: "1m", target: 100 }, // normal load
    { duration: "1m", target: 300 }, // stress
    { duration: "30s", target: 2000 }, // peak test
    { duration: "30s", target: 0 }, // cool down
  ],
  thresholds: {
    http_req_failed: ["rate<0.1"], // less than 10% failure allowed
    http_req_duration: ["p(95)<2000"], // 95% under 2s
  },
};

const BASE_URL = "https://kinlaybd.com";

export default function () {
  // 1. Homepage
  let home = http.get(`${BASE_URL}/`);
  check(home, {
    "homepage status 200": (r) => r.status === 200,
  });

  sleep(1);

  // 2. Product page simulation
  let products = http.get(`${BASE_URL}/checkout`);
  check(products, {
    "products load OK": (r) => r.status === 200,
  });

  sleep(1);

  // 3. Search simulation
  let search = http.get(`${BASE_URL}/?s=bag&post_type=product&product_cat=0`);
  check(search, {
    "search works": (r) => r.status === 200,
  });

  sleep(1);
}

// run the test
// k6 run kinlebd-load-test.js
// k6 run --out csv=results.csv kinlebd-load-test.js
// k6 run --out csv=results.csv kinlebd-load-test.js

```
