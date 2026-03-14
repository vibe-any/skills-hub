---
name: fetch-web-page-skill
description: Fetches and extracts text content from web pages when users need to read or analyze online articles, documentation, or web-based information
---

# Web Page Fetcher Skill

This skill enables Claude to retrieve and read content from web pages by extracting the main text from specified URLs. It's designed to help users access online information without leaving the conversation, making it easier to analyze articles, documentation, news, or any web-based content.

## Key Features
- Extracts clean, readable text content from web pages
- Removes navigation menus, ads, and other clutter automatically
- Handles various content types including articles, blog posts, and documentation
- Provides structured output with title and main content separated
- Works with most standard web pages and content management systems

## When to Use This Skill
Use this skill when you need to:
- Read and analyze online articles or blog posts
- Extract information from documentation websites
- Review news articles or press releases
- Access content from academic or research websites
- Analyze competitor websites or product pages
- Get information from any web page that contains primarily text content

## How to Use
1. Provide the complete URL of the web page you want to fetch
2. Specify if you need specific information or a general summary
3. Wait for the skill to extract and return the content
4. Ask follow-up questions about the extracted content as needed

## Examples
**Example 1: Reading an Article**
```
User: Please fetch and summarize the content from https://example.com/news/article-about-ai
Claude: I'll fetch that article for you and provide a summary of its key points.
```

**Example 2: Analyzing Documentation**
```
User: Can you read the API documentation at https://docs.example.com/api-reference and explain the authentication methods?
Claude: I'll retrieve the API documentation and focus on explaining the authentication methods for you.
```

**Example 3: Extracting Specific Information**
```
User: Fetch https://company.com/about-us and find their mission statement
Claude: I'll fetch the about page and locate their mission statement for you.
```

## Best Practices
- Always provide the complete URL including https:// or http://
- Be specific about what information you need from the page
- Allow time for the content to be fetched and processed
- Use this skill for legitimate research and information gathering purposes
- Consider that some websites may block automated access
- Verify important information by checking the original source when possible

## Limitations
- Cannot access pages behind paywalls or requiring login
- May not work with websites that have strong anti-bot protection
- Some dynamic content loaded by JavaScript may not be captured
- Cannot access local network addresses or localhost
- Rate limiting may apply to prevent abuse
- Content extraction quality varies depending on website structure
- Cannot handle file downloads (PDFs, images, videos) - only text content