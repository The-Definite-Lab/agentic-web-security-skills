---
name: performance-testing
description: >-
  Professional performance testing skill. Covers Core Web Vitals, Lighthouse
  audits, k6 load testing, stress testing, Firebase/Supabase query optimization,
  React performance profiling, Web API performance, WebPageTest, and
  performance budgets. Use when optimizing apps, before major launches,
  or after performance regressions are detected.
  Trigger phrases: "performance test", "load test", "slow", "lighthouse",
  "core web vitals", "LCP", "FCP", "k6", "stress test", "performance audit",
  "optimize", "page speed".
---

# PERFORMANCE TESTING — PROFESSIONAL SKILL

## SECTION 1: CORE WEB VITALS — MANDATORY METRICS

| Metric | Full Name | Good | Needs Work | Poor |
|---|---|---|---|---|
| **LCP** | Largest Contentful Paint | ≤2.5s | 2.5–4.0s | >4.0s |
| **FID/INP** | Input Delay / Interaction to Next Paint | ≤100ms | 100–300ms | >300ms |
| **CLS** | Cumulative Layout Shift | ≤0.1 | 0.1–0.25 | >0.25 |
| **FCP** | First Contentful Paint | ≤1.8s | 1.8–3.0s | >3.0s |
| **TTFB** | Time to First Byte | ≤800ms | 800ms–1.8s | >1.8s |

---

## SECTION 2: LIGHTHOUSE AUDIT (Quick — Run First)

```bash
# Install Lighthouse CLI
npm install -g lighthouse

# Run full audit
lighthouse https://yoursite.com \
  --output html \
  --output-path ./reports/lighthouse-report.html \
  --chrome-flags="--headless"

# Run on local dev server
lighthouse http://localhost:3000 \
  --output html \
  --output-path ./reports/lighthouse-local.html \
  --chrome-flags="--headless --no-sandbox"

# Specific categories
lighthouse https://yoursite.com \
  --only-categories=performance,accessibility,best-practices,seo \
  --output json \
  --output-path ./reports/lighthouse.json

# CI integration — fail below threshold
lighthouse https://yoursite.com \
  --output json \
  --output-path ./reports/lh.json \
  --chrome-flags="--headless"

# Parse score
node -e "
  const r = require('./reports/lh.json');
  const score = r.categories.performance.score * 100;
  console.log('Performance score:', score);
  if (score < 70) { console.error('FAIL: Below 70'); process.exit(1); }
"
```

---

## SECTION 3: k6 LOAD TESTING

### Install k6
```bash
# Windows (Chocolatey)
choco install k6

# Or download: https://k6.io/docs/get-started/installation/
```

### Basic Load Test Script
```javascript
// load-test.js
import http from 'k6/http';
import { check, sleep } from 'k6';
import { Rate, Trend } from 'k6/metrics';

const errorRate = new Rate('errors');
const responseTime = new Trend('response_time');

export const options = {
  stages: [
    { duration: '30s', target: 10 },   // Ramp up to 10 users
    { duration: '1m',  target: 50 },   // Stay at 50 users (normal load)
    { duration: '30s', target: 100 },  // Spike to 100 users
    { duration: '1m',  target: 100 },  // Stay at 100 users (stress)
    { duration: '30s', target: 0 },    // Ramp down
  ],
  thresholds: {
    http_req_duration: ['p(95)<500'],  // 95% of requests under 500ms
    http_req_failed:   ['rate<0.01'],  // Error rate under 1%
    errors:            ['rate<0.01'],
  },
};

export default function () {
  // Test homepage
  const homeRes = http.get('https://yoursite.com/');
  check(homeRes, {
    'homepage status 200': (r) => r.status === 200,
    'homepage loads fast': (r) => r.timings.duration < 1000,
  });
  errorRate.add(homeRes.status !== 200);
  responseTime.add(homeRes.timings.duration);

  // Test API endpoint
  const apiRes = http.post('https://yoursite.com/api/data', 
    JSON.stringify({ query: 'test' }),
    { headers: { 'Content-Type': 'application/json' } }
  );
  check(apiRes, {
    'API status 200': (r) => r.status === 200,
    'API response time OK': (r) => r.timings.duration < 300,
  });

  sleep(1);
}
```

### Run Load Tests
```bash
# Basic run
k6 run load-test.js

# With output
k6 run --out json=results.json load-test.js

# HTML report
k6 run --out web-dashboard load-test.js

# Smoke test (1 user, short duration — quick sanity check)
k6 run --vus 1 --duration 30s load-test.js

# Stress test (find breaking point)
k6 run --vus 200 --duration 5m load-test.js
```

### Firebase-Specific Load Test
```javascript
// firebase-load-test.js
import http from 'k6/http';
import { check, sleep } from 'k6';

const FIREBASE_PROJECT = 'your-project-id';
const BASE_URL = `https://firestore.googleapis.com/v1/projects/${FIREBASE_PROJECT}/databases/(default)/documents`;

export const options = {
  vus: 50,
  duration: '2m',
  thresholds: {
    http_req_duration: ['p(90)<1000'],  // Firestore can be slower
    http_req_failed: ['rate<0.01'],
  },
};

export default function () {
  // Test Firestore read (use test collection)
  const res = http.get(`${BASE_URL}/test-collection/test-doc`, {
    headers: { 'Authorization': `Bearer ${__ENV.FIREBASE_TOKEN}` }
  });
  
  check(res, {
    'Firestore read OK': (r) => r.status === 200,
    'Firestore response time OK': (r) => r.timings.duration < 1000,
  });
  
  sleep(0.5);
}

// Run: FIREBASE_TOKEN=xxx k6 run firebase-load-test.js
```

---

## SECTION 4: REACT PERFORMANCE OPTIMIZATION

### Profile with React DevTools
```javascript
// Enable profiling (development only)
import { Profiler } from 'react';

function onRenderCallback(id, phase, actualDuration, baseDuration) {
  if (actualDuration > 16) { // 16ms = 60fps threshold
    console.warn(`Slow render: ${id} took ${actualDuration}ms (${phase})`);
  }
}

<Profiler id="SlowComponent" onRender={onRenderCallback}>
  <SlowComponent />
</Profiler>
```

### Common Fixes
```javascript
// 1. Memoize expensive components
const ExpensiveList = React.memo(({ items }) => (
  <ul>{items.map(item => <li key={item.id}>{item.name}</li>)}</ul>
));

// 2. Memoize expensive calculations
const sortedItems = useMemo(() => 
  items.sort((a, b) => b.score - a.score), 
  [items]  // only recalculate when items change
);

// 3. Stable callback references
const handleClick = useCallback((id) => {
  setSelected(id);
}, []); // stable reference — no unnecessary re-renders

// 4. Lazy load heavy components
const HeavyChart = React.lazy(() => import('./HeavyChart'));

// 5. Virtualize long lists
import { FixedSizeList } from 'react-window';
// Instead of rendering 10,000 items, render only visible ones
```

### Bundle Size Analysis
```bash
# Analyze bundle size
npm install --save-dev webpack-bundle-analyzer

# For Vite (React + Vite projects)
npm install --save-dev rollup-plugin-visualizer

# vite.config.js
import { visualizer } from 'rollup-plugin-visualizer';
export default {
  plugins: [visualizer({ open: true })]  // opens report after build
}

npm run build  # generates stats.html

# Target: main bundle < 200KB gzipped
# Code split at route level
```

---

## SECTION 5: FIREBASE PERFORMANCE OPTIMIZATION

```javascript
// 1. Firestore — Efficient queries
// ❌ BAD: reads entire collection, filters client-side
const allUsers = await getDocs(collection(db, 'users'));
const filtered = allUsers.docs.filter(d => d.data().role === 'admin');

// ✅ GOOD: server-side filter with index
const admins = await getDocs(
  query(collection(db, 'users'), 
    where('role', '==', 'admin'),
    limit(50)  // always paginate
  )
);

// 2. Use compound indexes for complex queries
// firestore.indexes.json — define indexes for multi-field queries
{
  "indexes": [{
    "collectionGroup": "habits",
    "fields": [
      { "fieldPath": "userId", "order": "ASCENDING" },
      { "fieldPath": "createdAt", "order": "DESCENDING" }
    ]
  }]
}

// 3. Enable offline persistence (mobile + web)
import { enableIndexedDbPersistence } from 'firebase/firestore';
enableIndexedDbPersistence(db).catch(console.warn);

// 4. Batch writes (reduce network calls)
const batch = writeBatch(db);
items.forEach(item => batch.set(doc(db, 'items', item.id), item));
await batch.commit(); // one round trip instead of N

// 5. Real-time listener optimization — unsubscribe when done
const unsubscribe = onSnapshot(query, (snapshot) => { /* ... */ });
// In React useEffect cleanup:
return () => unsubscribe();
```

---

## SECTION 6: Web API performance,

```bash
# Enable Web API performance, Monitoring
# Admin > Server > Performance > Profiling

# Key settings to check:
# Admin > Development > Debugging — MUST be OFF in production
# Admin > Server > Caching — enable all caches (MUC)

# Database query analysis
# Enable slow query log in MySQL:
# slow_query_log = 1
# long_query_time = 1
# slow_query_log_file = /var/log/mysql/slow.log

# Check slow queries
mysqldumpslow -s t /var/log/mysql/slow.log | head -20

# Moodle-specific optimizations
# 1. Enable Boost caching in Admin > Site Admin > Development
# 2. Use Redis/Memcached for MUC (Moodle Universal Cache)
# 3. Enable opcache for PHP (php.ini):
#    opcache.enable=1
#    opcache.memory_consumption=256
#    opcache.max_accelerated_files=20000
```

---

## SECTION 7: WEBPAGETEST (Real-World Testing)

```bash
# Test from WebPageTest API
curl "https://www.webpagetest.org/runtest.php?url=https://yoursite.com&f=json&k=YOUR_API_KEY&runs=3&location=Dulles:Chrome"

# Or use PageSpeed Insights API
curl "https://www.googleapis.com/pagespeedonline/v5/runPagespeed?url=https://yoursite.com&strategy=mobile"

# Parse CWV scores
curl "https://www.googleapis.com/pagespeedonline/v5/runPagespeed?url=https://yoursite.com&strategy=mobile" | \
  python -c "
import json, sys
data = json.load(sys.stdin)
cats = data['lighthouseResult']['categories']
print(f\"Performance: {int(cats['performance']['score']*100)}\")
print(f\"SEO: {int(cats['seo']['score']*100)}\")
print(f\"Accessibility: {int(cats['accessibility']['score']*100)}\")
"
```

---

## SECTION 8: PERFORMANCE BUDGET (Targets per Project Type)

| Project | Performance | LCP | FCP | Bundle Size |
|---|---|---|---|---|
| Portfolio | ≥90 | <2s | <1.5s | <150KB gzip |
| Marketing site | ≥80 | <2.5s | <2s | <200KB gzip |
| Web App (React) | ≥70 | <3s | <2.5s | <400KB gzip |
| Moodle (LMS) | ≥60 | <4s | <3s | N/A |
| Mobile WebView | ≥75 | <3s | <2s | <300KB gzip |

---

## SECTION 9: PRE-LAUNCH PERFORMANCE CHECKLIST

```
Before Production Deploy:
├── [ ] Lighthouse score ≥ 70 on mobile
├── [ ] LCP < 2.5s
├── [ ] CLS < 0.1
├── [ ] Images: WebP format, lazy-loaded, sized correctly
├── [ ] JS bundle: code-split, no unused imports
├── [ ] CSS: purged (no unused CSS in production)
├── [ ] Fonts: preloaded, font-display: swap
├── [ ] Caching headers set (Cache-Control)
├── [ ] CDN configured for static assets
├── [ ] Gzip/Brotli compression enabled
├── [ ] k6 smoke test passed (1 user, 30s)
├── [ ] k6 load test passed (expected concurrent users, 2min)
└── [ ] Database queries: no N+1 queries, indexes verified
```
