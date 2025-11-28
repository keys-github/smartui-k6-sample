# SmartUI SDK Sample for K6

Welcome to the SmartUI SDK sample for K6. This repository demonstrates how to integrate SmartUI visual regression testing with K6 browser automation.

## Repository Structure

```
smartui-k6-sample/
├── k6-smartui.js              # Cloud test with single scenario
├── smartui-k6-scenarios.js    # Cloud test with multiple scenarios
└── smartui-web.json           # SmartUI config (create with npx smartui config:create)
```

## 1. Prerequisites and Environment Setup

### Prerequisites

- K6 installed (see [K6 Installation Guide](https://k6.io/docs/get-started/installation/))
- LambdaTest account credentials
- Node.js installed (for SmartUI CLI, optional)

### Install K6

**macOS:**
```bash
brew install k6
```

**Linux:**
```bash
sudo gpg -k
sudo gpg --no-default-keyring --keyring /usr/share/keyrings/k6-archive-keyring.gpg --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys C5AD17C747E3415A3642D57D77C6C491D6AC1D69
echo "deb [signed-by=/usr/share/keyrings/k6-archive-keyring.gpg] https://dl.k6.io/deb stable main" | sudo tee /etc/apt/sources.list.d/k6.list
sudo apt-get update
sudo apt-get install k6
```

**Windows:**
Download the installer from [K6 Installation Guide](https://k6.io/docs/get-started/installation/)

**Verify installation:**
```bash
k6 version
```

### Environment Setup

```bash
export LT_USERNAME='your_username'
export LT_ACCESS_KEY='your_access_key'
```

## 2. Initial Setup and Dependencies

### Clone the Repository

```bash
git clone https://github.com/LambdaTest/smartui-k6-sample
cd smartui-k6-sample
```

### Create SmartUI Configuration (Optional)

For K6 browser automation with hooks, the SmartUI config is optional. The `smartUIProjectName` in capabilities is used instead.

```bash
npx smartui config:create smartui-web.json
```

## 3. Steps to Integrate Screenshot Commands into Codebase

The SmartUI screenshot function is already implemented in the repository using hooks-based integration.

**Test File** (`k6-smartui.js`):
```javascript
import { chromium } from 'k6/experimental/browser';

await page.goto("https://www.lambdatest.com");

// Take screenshot using SmartUI hooks
await captureSmartUIScreenshot(page, "screenshot");

async function captureSmartUIScreenshot(page, screenshotName) {
  await page.evaluate(_ => {}, `lambdatest_action: ${JSON.stringify({ 
    action: "smartui.takeScreenshot", 
    arguments: { screenshotName: screenshotName } 
  })}`);
}
```

**Note**: 
- The code is already configured and ready to use
- You can modify the URL and screenshot name if needed
- Update `smartUIProjectName` in capabilities to match your SmartUI project name
- The test uses K6 browser automation with hooks-based SmartUI integration

## 4. Execution and Commands

Execute visual regression tests on SmartUI using the following command:

```bash
K6_BROWSER_ENABLED=true k6 run k6-smartui.js
```

**Note**: 
- `K6_BROWSER_ENABLED=true` is required to enable browser automation in K6
- Navigate to the [LambdaTest dashboard](https://automation.lambdatest.com/build) to view the running test
- Visit your SmartUI project to see captured screenshots

## Test Files

### Single Scenario Test (`k6-smartui.js`)

- Connects to LambdaTest Cloud using K6 browser automation
- Reads credentials from environment variables (`LT_USERNAME`, `LT_ACCESS_KEY`)
- Takes screenshot with name: `screenshot`
- Uses `smartui.takeScreenshot` action via hooks

### Multiple Scenarios Test (`smartui-k6-scenarios.js`)

- Demonstrates running tests with multiple browser scenarios
- Tests Chrome and Microsoft Edge browsers
- Uses per-VU iterations executor

## Configuration

### Capabilities

The test files include capabilities configuration for LambdaTest Cloud. Update `smartUIProjectName` to match your SmartUI project name:

```javascript
const capabilities = {
  "browserName": "Chrome",
  "browserVersion": "latest",
  "LT:Options": {
    "user": __ENV.LT_USERNAME,
    "accessKey": __ENV.LT_ACCESS_KEY,
    "smartUIProjectName": "K6_Test_Sample",  // Update this to match your project
  }
};
```

## Best Practices

### Screenshot Naming

- Use descriptive, unique names for each screenshot
- Include scenario and state information
- Avoid special characters
- Use consistent naming conventions

### When to Take Screenshots

- After page loads
- After user interactions
- At different stages of user journeys
- After state changes

### K6-Specific Tips

- Always set `K6_BROWSER_ENABLED=true` environment variable
- Use `page.waitForLoadState()` before screenshots
- Handle errors gracefully in scenarios
- Use proper VU (Virtual User) configuration

### Example: Screenshot with Wait

```javascript
import { chromium } from 'k6/experimental/browser';

export default async function () {
  const browser = chromium.launch();
  const page = browser.newPage();
  
  await page.goto('https://www.lambdatest.com');
  await page.waitForLoadState('networkidle');
  
  await captureSmartUIScreenshot(page, "homepage");
  
  browser.close();
}
```

## Common Use Cases

### Multiple Scenarios

```javascript
import { chromium } from 'k6/experimental/browser';

export const options = {
  scenarios: {
    ui: {
      executor: 'shared-iterations',
      options: {
        browser: {
          type: 'chromium',
        },
      },
    },
  },
};

export default async function () {
  const browser = chromium.launch();
  const page = browser.newPage();
  
  await page.goto('https://www.lambdatest.com');
  await captureSmartUIScreenshot(page, "homepage");
  
  browser.close();
}
```

### Performance Testing with Visual Validation

```javascript
import { check } from 'k6';
import { chromium } from 'k6/experimental/browser';

export default async function () {
  const browser = chromium.launch();
  const page = browser.newPage();
  
  const response = await page.goto('https://www.lambdatest.com');
  check(response, {
    'status is 200': (r) => r.status() === 200,
  });
  
  await captureSmartUIScreenshot(page, "homepage-performance-test");
  
  browser.close();
}
```

## CI/CD Integration

### GitHub Actions Example

```yaml
name: K6 SmartUI Tests

on: [push, pull_request]

jobs:
  visual-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Install K6
        run: |
          sudo gpg -k
          sudo gpg --no-default-keyring --keyring /usr/share/keyrings/k6-archive-keyring.gpg --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys C5AD17C747E3415A3642D57D77C6C491D6AC1D69
          echo "deb [signed-by=/usr/share/keyrings/k6-archive-keyring.gpg] https://dl.k6.io/deb stable main" | sudo tee /etc/apt/sources.list.d/k6.list
          sudo apt-get update
          sudo apt-get install k6
      
      - name: Run K6 with SmartUI
        env:
          K6_BROWSER_ENABLED: true
          LT_USERNAME: ${{ secrets.LT_USERNAME }}
          LT_ACCESS_KEY: ${{ secrets.LT_ACCESS_KEY }}
        run: |
          k6 run k6-smartui.js
```

## Troubleshooting

### Issue: `K6_BROWSER_ENABLED is not set`

**Solution**: Always set the environment variable:
```bash
export K6_BROWSER_ENABLED=true
k6 run k6-smartui.js
```

### Issue: Screenshots not captured

**Solution**:
1. Verify `LT_USERNAME` and `LT_ACCESS_KEY` are set
2. Check `smartUIProjectName` in capabilities matches your project
3. Ensure K6 browser automation is enabled
4. Wait for page to load before taking screenshots

### Issue: `Unauthorized` error

**Solution**:
1. Verify LambdaTest credentials are correct
2. Check credentials in [LambdaTest Profile Settings](https://accounts.lambdatest.com/profile)
3. Ensure no extra spaces in environment variables

### Issue: Browser automation not working

**Solution**:
1. Verify K6 version supports browser automation (v0.43.0+)
2. Ensure `K6_BROWSER_ENABLED=true` is set
3. Check K6 installation: `k6 version`

## Configuration Tips

### Capabilities Configuration

Update capabilities in your test file:
```javascript
const capabilities = {
  "browserName": "Chrome",
  "browserVersion": "latest",
  "LT:Options": {
    "user": __ENV.LT_USERNAME,
    "accessKey": __ENV.LT_ACCESS_KEY,
    "platform": "Windows 10",
    "smartUIProjectName": "Your_Project_Name",  // Update this
    "smartUIBaseline": false  // Set to false to update baseline
  }
};
```

### VU Configuration

Configure virtual users appropriately:
```javascript
export const options = {
  vus: 1,  // Number of virtual users
  iterations: 1,  // Number of iterations
  scenarios: {
    ui: {
      executor: 'shared-iterations',
      options: {
        browser: {
          type: 'chromium',
        },
      },
    },
  },
};
```

## View Results

After running the tests, visit your [LambdaTest dashboard](https://automation.lambdatest.com/build) to view the running test and SmartUI project to see captured screenshots.

## Additional Resources

- [SmartUI K6 Onboarding Guide](https://www.lambdatest.com/support/docs/smartui-onboarding-k6/)
- [K6 Documentation](https://k6.io/docs/)
- [K6 Browser Automation](https://k6.io/docs/using-k6/browser/)
- [LambdaTest K6 Documentation](https://www.lambdatest.com/support/docs/k6-testing/)
- [SmartUI Dashboard](https://smartui.lambdatest.com/)
- [LambdaTest Community](https://community.lambdatest.com/)

## Notes

- K6 browser automation requires `K6_BROWSER_ENABLED=true` environment variable
- The repository uses hooks-based SmartUI integration (not SDK-based)
- Screenshots are captured using `smartui.takeScreenshot` action via `page.evaluate()`
- K6 browser automation is available in K6 v0.43.0 and later
