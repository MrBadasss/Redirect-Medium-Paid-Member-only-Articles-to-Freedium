# Medium to Freedium Redirect

A Tampermonkey userscript that automatically redirects Medium articles to Freedium when the article is marked as "Member-only."

## Features
- **Automatic redirection**: Detects Medium articles and checks if they are "Member-only."
- **Real-time detection**: Uses a DOM Mutation Observer to catch dynamically loaded "Member-only" content.
- **Fast and efficient**: Redirects as soon as the "Member-only" string is found.

## Installation

1. Install [Tampermonkey](https://www.tampermonkey.net/) for your browser.
2. Create a new userscript in Tampermonkey.
3. Copy and paste the following script into the new userscript file.

```javascript
// ==UserScript==
// @name         Redirect Medium Paid (Member-only) Articles to Freedium
// @namespace    https://github.com/MrBadasss/Redirect-Medium-Paid-Member-only-Articles-to-Freedium/
// @version      1.0
// @description  Redirects to Freedium if "Member-only" is found in Medium articles on first visit
// @author       Mr_Badasss
// @match        *://*/*
// @icon         https://www.google.com/s2/favicons?sz=64&domain=freedium.cfd
// @run-at       document-start
// @grant        none
// ==/UserScript==

const isMediumArticle = () => !!document.querySelector(`meta[property="al:android:url"][content^="medium://p/"]`)?.content;

const containsMemberOnly = () => document.body.innerText.includes("Member-only");

// Function to check the page for "Member-only" and redirect if found
const checkAndRedirect = (observer) => {
    if (containsMemberOnly()) {
        window.location.href = `https://freedium.cfd/` + encodeURIComponent(window.location.href);
        if (observer) observer.disconnect(); // Stop observing once redirected
    }
};

// Start observing the page for changes
new MutationObserver((mutations, observer) => {
    if (isMediumArticle()) {
        checkAndRedirect(observer);  // Check immediately in case "Member-only" is already present
        
        // Continue checking for future changes in the DOM (dynamic content)
        for (const mutation of mutations) {
            for (const node of mutation.addedNodes) {
                if (node.nodeType === Node.ELEMENT_NODE) {
                    checkAndRedirect(observer);
                }
            }
        }
    }
}).observe(document, { subtree: true, childList: true });

// Perform an initial check after the page has fully loaded
window.addEventListener('load', () => {
    if (isMediumArticle()) {
        setTimeout(() => {
            checkAndRedirect();  // Final check to ensure redirection occurs if "Member-only" is detected after load
        }, 1000); // Allow for additional delay if needed for page content to render
    }
});
