const express = require('express');
const cors = require('cors');
const axios = require('axios');
const cheerio = require('cheerio');
const path = require('path');

const app = express();
const PORT = process.env.PORT || 3000;

// Middleware
app.use(cors()); // Allow frontend to communicate
app.use(express.static('.')); // Serve static files from current directory

// Ignore favicon requests
app.get('/favicon.ico', (req, res) => res.status(204));

// Random User Agents to mimic real browsers
const userAgents = [
    'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36',
    'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/14.1.1 Safari/605.1.15',
    'Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/92.0.4515.107 Safari/537.36'
];

// Helper function to extract number from string
function extractResultCount(text) {
    if (!text) return 0;
    // Look for patterns like "About 1,200 results" or "6 results"
    const match = text.match(/([\d,]+)\s+results/);
    if (match && match[1]) {
        return parseInt(match[1].replace(/,/g, ''), 10);
    }
    // Sometimes it just says "10 results" or similar without "About"
    const simpleMatch = text.match(/^([\d,]+)\s+results/);
    if (simpleMatch && simpleMatch[1]) {
        return parseInt(simpleMatch[1].replace(/,/g, ''), 10);
    }
    return 0;
}

// API Endpoint
app.get('/api/check', async (req, res) => {
    const keyword = req.query.q;

    if (!keyword) {
        return res.status(400).json({ error: 'Keyword is required' });
    }

    console.log(`Checking allintitle for: ${keyword}`);

    try {
        // Construct Google Search URL with allintitle operator
        const searchUrl = `https://www.google.com/search?q=allintitle:${encodeURIComponent(keyword)}&hl=en`;
        
        // Randomize User Agent
        const userAgent = userAgents[Math.floor(Math.random() * userAgents.length)];

        const response = await axios.get(searchUrl, {
            headers: {
                'User-Agent': userAgent,
                'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8',
                'Accept-Language': 'en-US,en;q=0.5',
                'Referer': 'https://www.google.com/'
            }
        });

        const $ = cheerio.load(response.data);
        
        // Google's result stats div often has ID 'result-stats'
        let resultStats = $('#result-stats').text();
        
        let count = 0;

        // If result-stats is found, parse it
        if (resultStats) {
            console.log(`Found stats: ${resultStats}`);
            count = extractResultCount(resultStats);
        } else {
            // Sometimes if there are 0 results, Google shows a different message like "did not match any documents"
            const noResults = $('div:contains("did not match any documents")').length > 0;
            if (noResults) {
                count = 0;
            } else {
                // If we get here, likely blocked or layout changed
                console.log('Could not find result stats. Layout might be different or CAPTCHA triggered.');
                // For the sake of the user not seeing an error immediately if blocked, 
                // we might want to return a specific error or 0.
                // Let's try to detect CAPTCHA
                if ($('form[action="CaptchaRedirect"]').length > 0 || response.data.includes('recaptcha')) {
                    return res.status(429).json({ error: 'Google CAPTCHA triggered. Please try again later or use an API key.' });
                }
                
                // Fallback: Check if it's just very few results where result-stats is missing
                // Sometimes for < 10 results, it just lists them.
                const results = $('div.g').length;
                if (results > 0 && results < 20) {
                    count = results;
                }
            }
        }

        return res.json({ 
            keyword, 
            count,
            formatted: count.toLocaleString() 
        });

    } catch (error) {
        console.error('Error fetching data:', error.message);
        return res.status(500).json({ error: 'Failed to fetch results. Google might be blocking requests.' });
    }
});

app.listen(PORT, () => {
    console.log(`Server running at http://localhost:${PORT}`);
});
