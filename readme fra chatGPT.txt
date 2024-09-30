To achieve a fully server-side solution where the client doesn't load the actual ad unit or render the creative, you need to handle the ad auction, fetch the creative data, and render the ad server-side. This approach keeps the ad logic away from the client, preventing them from loading or rendering the ad directly.

Here's the general plan for setting this up:

Step 1: Server-Side Fetching of the Ad Unit
You need to move the logic of fetching the ad unit (which includes ad slot configuration and creative rendering) to the server side. To do this, you’ll use the Google Ad Manager API to request the ad and retrieve the creative as a JSON response. The server will then handle injecting the creative into the response sent to the client.

Server-Side Steps:

Make a request to Google Ad Manager (GAM) API or any external demand-side platforms (DSPs) you are using to fetch ad auctions and creatives.
Parse the response, which includes the ad creative (usually in JSON format).
Inject the ad creative (like HTML or assets) into your server-rendered page.
Step 2: GAM API Integration on the Server-Side
You will use the Google Ad Manager API to fetch the ads. Here's a conceptual approach using a server (e.g., Node.js):

Example of Server-Side Ad Fetching using Node.js
javascript
Kopier kode
const fetch = require('node-fetch');  // You can also use any other HTTP client, e.g., axios

// Function to fetch ad creative from GAM
async function fetchAdFromGAM() {
  try {
    // Make a request to GAM (you'll need to use the GAM API with authentication)
    const response = await fetch(
      'https://securepubads.g.doubleclick.net/gampad/adx?iu=/6355419/Travel/Europe/France/Paris&sz=300x250&output=json',
      {
        method: 'GET',
        headers: {
          'Content-Type': 'application/json',
          // Add any necessary authentication headers (OAuth, API Key, etc.)
        },
      }
    );
    
    // Parse the response
    const adData = await response.json();

    // This will include ad creative details
    return adData;  // Return the ad creative JSON or HTML snippet
  } catch (err) {
    console.error('Error fetching ad from GAM:', err);
    return null;
  }
}
Step 3: Inject the Ad Creative into the HTML
Once you have fetched the ad creative, you can inject the returned HTML or ad details directly into your server-side rendered HTML.

Here’s a conceptual approach:

Example of Injecting Ad Creative in Node.js
javascript
Kopier kode
const express = require('express');
const app = express();
const port = 3000;

// Mock function to get ad creative
async function fetchAdCreative() {
  const adData = await fetchAdFromGAM();  // Fetch ad data from GAM
  // Assume adData contains a field `htmlSnippet` with the ad creative.
  const adCreative = adData && adData.htmlSnippet ? adData.htmlSnippet : 'No Ad Available';
  return adCreative;
}

app.get('/', async (req, res) => {
  const adCreative = await fetchAdCreative();  // Fetch ad creative from the server-side

  // Inject the creative into the server-side HTML
  const htmlContent = `
    <!doctype html>
    <html>
      <head>
        <meta charset="utf-8" />
        <meta name="viewport" content="width=device-width, initial-scale=1" />
        <title>Server-Side Ad Example</title>
      </head>
      <body>
        <div id="banner-ad">
          ${adCreative}  <!-- Inject the ad creative here -->
        </div>
      </body>
    </html>
  `;

  res.send(htmlContent);  // Serve the HTML with the ad
});

app.listen(port, () => {
  console.log(`Server running on http://localhost:${port}`);
});
Step 4: Prevent Client-Side Rendering
With this setup:

The ad unit is not fetched or rendered on the client-side. The server does all the heavy lifting.
The ad creative is injected into the HTML server-side and delivered as part of the response.
Step 5: Ad Auction Configuration
If you’re handling multiple ad units or an auction, you can integrate Open Bidding with the Google Ad Manager API or a header bidding service like Prebid.js (server-side). This will allow you to decide which creative to render after conducting the auction, but still, everything happens on the server.

Summary
Server makes the request: You request ad auctions and creatives from Google Ad Manager or a demand-side platform.
Server processes the response: After fetching the response, the server parses it, extracts the ad creative (usually HTML or JavaScript).
Inject creative server-side: The server renders the creative into the page that’s delivered to the client, so the client never directly interacts with Google Ad Manager.
This approach ensures that ads are fetched and rendered entirely on the server with no exposure to the ad auction process on the client side.