# 📋 COPY THIS CODE TO CLOUDFLARE WORKER

## Instructions:
1. Go to: https://dash.cloudflare.com/
2. Click: Workers & Pages → Create application → Create Worker
3. Name it: epicvin-form-emailer
4. Click: Edit code
5. DELETE ALL existing code
6. COPY everything below this line
7. PASTE into the worker editor
8. Click: Save and Deploy
9. COPY your worker URL (you'll need it for index.html)

---

```javascript
/**
 * Cloudflare Worker - Email Form Entries
 * Sends form data to: car.check.store@gmail.com
 * Before Paddle checkout opens
 */

export default {
  async fetch(request, env, ctx) {
    // Handle CORS preflight
    if (request.method === 'OPTIONS') {
      return new Response(null, {
        headers: {
          'Access-Control-Allow-Origin': '*',
          'Access-Control-Allow-Methods': 'POST, OPTIONS',
          'Access-Control-Allow-Headers': 'Content-Type',
        },
      });
    }

    // Only POST requests
    if (request.method !== 'POST') {
      return new Response(JSON.stringify({ error: 'Method not allowed' }), {
        status: 405,
        headers: {
          'Content-Type': 'application/json',
          'Access-Control-Allow-Origin': '*',
        },
      });
    }

    try {
      const data = await request.json();
      const { vin, plate, state, vehicleType, searchType, timestamp } = data;

      // Validate input
      if (!vin && !plate) {
        return new Response(JSON.stringify({ error: 'VIN or Plate required' }), {
          status: 400,
          headers: {
            'Content-Type': 'application/json',
            'Access-Control-Allow-Origin': '*',
          },
        });
      }

      // Email subject
      const emailSubject = `New ${searchType === 'vin' ? 'VIN' : 'License Plate'} Check Request`;
      
      // Email HTML body
      let emailBody = `
<!DOCTYPE html>
<html>
<head>
  <style>
    body { font-family: Arial, sans-serif; line-height: 1.6; color: #333; }
    .container { max-width: 600px; margin: 0 auto; padding: 20px; }
    .header { background-color: #0084ff; color: white; padding: 20px; text-align: center; }
    .content { background-color: #f9f9f9; padding: 20px; margin-top: 20px; border-radius: 5px; }
    .field { margin-bottom: 15px; }
    .label { font-weight: bold; color: #0084ff; }
    .value { color: #333; padding: 5px 0; }
    .footer { margin-top: 20px; padding-top: 20px; border-top: 1px solid #ddd; font-size: 12px; color: #666; }
  </style>
</head>
<body>
  <div class="container">
    <div class="header">
      <h1>🚗 New Vehicle Check Request</h1>
    </div>
    <div class="content">
      <h2>Vehicle Information</h2>
`;

      if (vin) {
        emailBody += `
      <div class="field">
        <div class="label">VIN Number:</div>
        <div class="value">${vin}</div>
      </div>
`;
      }

      if (plate) {
        emailBody += `
      <div class="field">
        <div class="label">License Plate:</div>
        <div class="value">${plate}</div>
      </div>
`;
      }

      if (state) {
        emailBody += `
      <div class="field">
        <div class="label">State:</div>
        <div class="value">${state}</div>
      </div>
`;
      }

      if (vehicleType) {
        emailBody += `
      <div class="field">
        <div class="label">Vehicle Type:</div>
        <div class="value">${vehicleType}</div>
      </div>
`;
      }

      emailBody += `
      <div class="field">
        <div class="label">Search Type:</div>
        <div class="value">${searchType || 'N/A'}</div>
      </div>
`;

      if (timestamp) {
        emailBody += `
      <div class="field">
        <div class="label">Timestamp:</div>
        <div class="value">${new Date(timestamp).toLocaleString()}</div>
      </div>
`;
      }

      emailBody += `
    </div>
    <div class="footer">
      <p>This email was automatically generated from EpicVIN vehicle history check form.</p>
    </div>
  </div>
</body>
</html>
`;

      // Send via Mailchannels (FREE)
      const emailResponse = await fetch('https://api.mailchannels.net/tx/v1/send', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({
          personalizations: [
            {
              to: [{ email: 'car.check.store@gmail.com', name: 'EpicVIN Admin' }],
            },
          ],
          from: {
            email: 'noreply@epicvinrecord.com',
            name: 'EpicVIN Report',
          },
          subject: emailSubject,
          content: [
            {
              type: 'text/html',
              value: emailBody,
            },
          ],
        }),
      });

      // Return success (even if email fails, don't block checkout)
      if (emailResponse.ok) {
        return new Response(JSON.stringify({ 
          success: true, 
          message: 'Form data sent successfully'
        }), {
          status: 200,
          headers: {
            'Content-Type': 'application/json',
            'Access-Control-Allow-Origin': '*',
          },
        });
      } else {
        console.error('Email send failed:', await emailResponse.text());
        return new Response(JSON.stringify({ 
          success: true,
          message: 'Request processed'
        }), {
          status: 200,
          headers: {
            'Content-Type': 'application/json',
            'Access-Control-Allow-Origin': '*',
          },
        });
      }

    } catch (error) {
      console.error('Worker error:', error);
      return new Response(JSON.stringify({ 
        success: true, 
        message: 'Request processed' 
      }), {
        status: 200,
        headers: {
          'Content-Type': 'application/json',
          'Access-Control-Allow-Origin': '*',
        },
      });
    }
  },
};
```

---

## ✅ After Deploying:

Your worker URL will look like:
```
https://epicvin-form-emailer.YOUR-NAME.workers.dev/
```

## 📝 Next Step:

Update this line in `index.html`:
```javascript
const WORKER_URL = 'YOUR-WORKER-URL-HERE';
```

Replace with your actual worker URL!

## 🎉 Done!

Form entries will now be emailed to **car.check.store@gmail.com** before Paddle checkout!
