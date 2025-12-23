# Email Form Integration with Cloudflare Workers

This integration sends form entries to `car.check.store@gmail.com` before opening the Paddle checkout.

## 🚀 Setup Instructions

### Step 1: Deploy the Cloudflare Worker

1. **Log in to Cloudflare Dashboard**
   - Go to https://dash.cloudflare.com/
   - Sign in to your account (or create one if you don't have it)

2. **Navigate to Workers & Pages**
   - In the left sidebar, click on "Workers & Pages"
   - Click the "Create application" button
   - Select "Create Worker"

3. **Create the Worker**
   - Give your worker a name (e.g., `epicvin-form-emailer`)
   - Click "Deploy"

4. **Edit the Worker Code**
   - After deployment, click "Edit code"
   - Delete all the default code
   - Copy the entire content from `cloudflare-worker.js` file
   - Paste it into the worker editor
   - Click "Save and Deploy"

5. **Copy the Worker URL**
   - After deploying, you'll see your worker URL
   - It will look like: `https://epicvin-form-emailer.YOUR-SUBDOMAIN.workers.dev/`
   - Copy this URL - you'll need it for the next step

### Step 2: Update the Website

1. **Open the index.html file**
   - Find line with: `const WORKER_URL = 'https://cold-hat-5fd3.rmoto7817.workers.dev/';`
   - Replace the URL with your actual worker URL from Step 1

2. **Save the file**
   - The integration is now complete!

## 📧 How It Works

1. **User fills the form** (VIN or License Plate)
2. **User clicks the "Check Report" button**
3. **Form data is sent to Cloudflare Worker** (via the WORKER_URL)
4. **Worker sends email** to car.check.store@gmail.com
5. **Paddle checkout opens** (user continues with payment)

The email sending happens **in the background** and won't block the user's checkout experience.

## 📝 Email Content

The email will contain:
- **VIN Number** (if provided)
- **License Plate** (if provided)
- **State** (for plate searches)
- **Vehicle Type** (sedan, suv, truck, etc.)
- **Search Type** (vin or plate)
- **Timestamp** (when the form was submitted)

### Sample Email

```
Subject: New VIN Check Request

🚗 New Vehicle Check Request

Vehicle Information

VIN Number: 1HGBH41JXMN109186
Vehicle Type: sedan
Search Type: vin
Timestamp: 12/23/2025, 2:30:45 PM
```

## 🔧 Technical Details

### Email Service

The worker uses **Mailchannels**, which is:
- ✅ **Free** (no cost)
- ✅ **Integrated** with Cloudflare Workers
- ✅ **Reliable** (high deliverability)
- ✅ **No API Key Required**

### CORS Configuration

The worker is configured to accept requests from any domain:
```javascript
'Access-Control-Allow-Origin': '*'
```

If you want to restrict it to only your domain, change it to:
```javascript
'Access-Control-Allow-Origin': 'https://epicvinrecord.com'
```

## 🐛 Troubleshooting

### Emails Not Arriving

1. **Check Spam Folder**
   - Mailchannels emails might go to spam initially
   - Mark them as "Not Spam" to train the filter

2. **Check Worker Logs**
   - Go to your worker in Cloudflare Dashboard
   - Click "Logs" tab
   - Look for any error messages

3. **Verify Worker URL**
   - Make sure the `WORKER_URL` in index.html matches your actual worker URL
   - Check for typos or missing slashes

### Worker Not Responding

1. **Check Worker Status**
   - Go to Cloudflare Dashboard
   - Verify the worker is deployed and active

2. **Test the Worker**
   - Open browser console (F12)
   - Go to Network tab
   - Submit a form and check if the request to worker URL succeeds

## 🔒 Security Notes

- The worker doesn't store any data
- All data is sent directly to the email
- HTTPS is used for all communications
- No sensitive payment data is transmitted to the worker

## 📊 Monitoring

To monitor form submissions:
1. Check the email inbox: car.check.store@gmail.com
2. View Cloudflare Worker logs for request count and errors

## 💡 Customization

### Change Email Address

Edit line in `cloudflare-worker.js`:
```javascript
to: [{ email: 'car.check.store@gmail.com', name: 'EpicVIN Admin' }],
```

### Change Email Template

Edit the `emailBody` variable in `cloudflare-worker.js` to customize:
- Subject line
- HTML styling
- Content layout
- Additional fields

### Add More Form Fields

In `index.html`, add fields to the `formData` object:
```javascript
const formData = {
    vin: vin || null,
    plate: plate || null,
    state: state || null,
    vehicleType: vehicleType || 'sedan',
    searchType: searchType || 'vin',
    timestamp: new Date().toISOString(),
    // Add your custom fields here
    customField: 'value'
};
```

Then update the worker to include these fields in the email.

## 📦 Files Included

1. **cloudflare-worker.js** - The Cloudflare Worker code
2. **index.html** - Updated with worker integration
3. **SETUP_GUIDE.md** - This file

## ✅ Testing

1. **Fill out a form** on your website
2. **Click "Check Report"** button
3. **Check email** at car.check.store@gmail.com
4. **Verify** Paddle checkout still opens normally

## 🎉 Success!

If you receive an email after submitting the form, the integration is working correctly!

---

**Need Help?** 
- Check Cloudflare Workers documentation: https://developers.cloudflare.com/workers/
- Review Mailchannels API: https://mailchannels.zendesk.com/hc/en-us/articles/4565898358413
