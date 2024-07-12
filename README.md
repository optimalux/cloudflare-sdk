# Optimal UX Agent for Cloudflare

This agent should be installed in your Cloudflare account to enable the Optimal UX experiments.
The agent will download configuration from https://api.optimalux.com and replace content on your website according to
the configured experiments. It also injects a tracking JavaScript to collect data on experiment success.
The agent may adjust Content-Security-Policy header to allow the tracking script to run.

## Installation

It is recommended to use Optimal UX interface to install the agent. [Read more](https://optimalux.com/documentation/cloudflare-token-for-automatic-installation).

You can include it into your existing worker but installing the dependency:
`yarn add @optimalux/cloudflare-sdk`

Include the function into your worker script like this:

```javascript
import { fetchWithExperiments } from '@optimalux/cloudflare-sdk';

export default {
    async fetch(request, env, ctx) {
        return fetchWithExperiments(request, env, ctx); 
    }
}
```

## Configuration

The agent requires the following environment variables to be set in your Cloudflare account
which are passed as `env` parameter to the `fetchWithExperiments` function:

-   `OPTIMALUX_SITE_ID` - Site ID to access the Optimal UX API
-   `OPTIMALUX_API_KEY` - API key to access the Optimal UX API

## Advanced usage
For more granular control over the request flow (e.g. for caching),
you may need to call Optimal UX functions in different places in your worker script.

Make sure to include `handleOptuxPages` function in the beginning of the request handler:

```javascript
import { handleOptuxPages } from '@optimalux/cloudflare-sdk';

export default {
    async fetch(request, env, ctx) {
        // This block should be in the beginning of your fetch handler
        const optux = new OptimaluxCloudflareWorker(request, env, ctx);
        if (optux.isSpecialPage()) {
            return optux.serveSpecialPage(request, env, ctx);
        }
        
        // Fetch content or retrieve it from cache
        const response = await fetch(optux.getRequest());
        
        // Update response with running experiments
        return optux.handleResponse(response);
        
    }
}
```
