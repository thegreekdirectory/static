// netlify/functions/upload.js
// This handles uploads to GitHub on behalf of users

const fetch = require('node-fetch');

exports.handler = async (event, context) => {
  // Only allow POST requests
  if (event.httpMethod !== 'POST') {
    return {
      statusCode: 405,
      body: JSON.stringify({ error: 'Method not allowed' })
    };
  }

  // Enable CORS
  const headers = {
    'Access-Control-Allow-Origin': '*',
    'Access-Control-Allow-Headers': 'Content-Type',
    'Access-Control-Allow-Methods': 'POST, OPTIONS',
    'Content-Type': 'application/json'
  };

  // Handle preflight
  if (event.httpMethod === 'OPTIONS') {
    return { statusCode: 200, headers, body: '' };
  }

  try {
    // Parse the request body
    const { brandName, fileName, fileContent } = JSON.parse(event.body);

    // Validate inputs
    if (!brandName || !fileName || !fileContent) {
      return {
        statusCode: 400,
        headers,
        body: JSON.stringify({ error: 'Missing required fields' })
      };
    }

    // Validate brand name (lowercase, alphanumeric, hyphens only)
    if (!/^[a-z0-9-]+$/.test(brandName)) {
      return {
        statusCode: 400,
        headers,
        body: JSON.stringify({ error: 'Brand name must be lowercase alphanumeric with hyphens only' })
      };
    }

    // GitHub configuration - STORE THESE AS ENVIRONMENT VARIABLES
    const GITHUB_TOKEN = process.env.GITHUB_TOKEN;
    const GITHUB_USERNAME = process.env.GITHUB_USERNAME;
    const REPO_NAME = 'static';

    if (!GITHUB_TOKEN || !GITHUB_USERNAME) {
      return {
        statusCode: 500,
        headers,
        body: JSON.stringify({ error: 'Server configuration error' })
      };
    }

    // GitHub API endpoint
    const apiUrl = `https://api.github.com/repos/${GITHUB_USERNAME}/${REPO_NAME}/contents/${brandName}/${fileName}`;

    // Check if file already exists
    let sha = null;
    try {
      const checkResponse = await fetch(apiUrl, {
        headers: {
          'Authorization': `token ${GITHUB_TOKEN}`,
          'Accept': 'application/vnd.github.v3+json',
          'User-Agent': 'StaticMediaUploader'
        }
      });
      
      if (checkResponse.ok) {
        const data = await checkResponse.json();
        sha = data.sha;
      }
    } catch (e) {
      // File doesn't exist, continue
    }

    // Upload file to GitHub
    const uploadResponse = await fetch(apiUrl, {
      method: 'PUT',
      headers: {
        'Authorization': `token ${GITHUB_TOKEN}`,
        'Accept': 'application/vnd.github.v3+json',
        'Content-Type': 'application/json',
        'User-Agent': 'StaticMediaUploader'
      },
      body: JSON.stringify({
        message: `Upload ${fileName} for ${brandName}`,
        content: fileContent, // Already base64 encoded
        sha: sha // Include if updating existing file
      })
    });

    if (!uploadResponse.ok) {
      const error = await uploadResponse.json();
      throw new Error(error.message || 'GitHub upload failed');
    }

    const data = await uploadResponse.json();

    return {
      statusCode: 200,
      headers,
      body: JSON.stringify({
        success: true,
        url: `https://static.thegreekdirectory.org/${brandName}/${fileName}`,
        message: 'File uploaded successfully'
      })
    };

  } catch (error) {
    console.error('Upload error:', error);
    return {
      statusCode: 500,
      headers,
      body: JSON.stringify({ 
        error: 'Upload failed', 
        message: error.message 
      })
    };
  }
};
