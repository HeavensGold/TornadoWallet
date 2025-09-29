# QR Code Fix - Replace Deprecated Google Charts API

## Problem
The QR code generation in the TornadoWallet was using the deprecated Google Charts API:
```html
<img src="http://chart.apis.google.com/chart?cht=qr&chs=300x300&chl={{ address }}&chld=H|0"/>
```

This API has been deprecated and no longer works, causing QR codes to fail to display.

## Solution
Replaced the deprecated Google Charts API with a modern client-side QR code generation solution using the `qrcode-generator` library.

## Changes Made

### 1. Downloaded QR Code Library
```bash
curl -o /home/od41/git/TornadoWallet/wallet/themes/common/js/qrcode.min.js \
  https://cdnjs.cloudflare.com/ajax/libs/qrcode-generator/1.4.4/qrcode.min.js
```

### 2. Updated Template: `wallet/themes/material/transactions_receive.html`

**Before:**
```html
<img src="http://chart.apis.google.com/chart?cht=qr&chs=300x300&chl={{ address }}&chld=H|0"/>
```

**After:**
```html
<div id="qrcode-address" style="text-align: center; margin: 20px 0;"></div>
```

**Before:**
```html
<img src="http://chart.apis.google.com/chart?cht=qr&chs=300x300&chl={{ bisurl }}&chld=H|0"/>
```

**After:**
```html
<div id="qrcode-bisurl" style="text-align: center; margin: 20px 0;"></div>
```

### 3. Added JavaScript for QR Generation
```html
<script src="/common/js/qrcode.min.js"></script>
<script>
document.addEventListener('DOMContentLoaded', function() {
    function generateQR(text, elementId) {
        try {
            var qr = qrcode(0, 'H');
            qr.addData(text);
            qr.make();
            
            var qrHTML = qr.createImgTag(4, 8);
            document.getElementById(elementId).innerHTML = qrHTML;
        } catch (error) {
            console.error('QR Code generation failed:', error);
            document.getElementById(elementId).innerHTML = '<p>QR Code generation failed</p>';
        }
    }
    
    // Generate QR code for address
    const address = "{{ address }}";
    if (address) {
        generateQR(address, 'qrcode-address');
    }
    
    // Generate QR code for BIS URL if available
    {% if bisurl %}
    const bisurl = "{{ bisurl }}";
    if (bisurl) {
        generateQR(bisurl, 'qrcode-bisurl');
    }
    {% end %}
});
</script>
```

## Technical Details

### Static File Routing
The application routes `/common/(.*)` to the `themes/common` directory, so the QR code library is accessible at `/common/js/qrcode.min.js`.

### Library Choice
- **qrcode-generator**: Lightweight, reliable QR code generation library
- **Client-side**: No server dependencies required
- **Error correction**: Uses 'H' (high) error correction level
- **Size**: Generates images similar to the original 300x300 size

## Benefits
1. ✅ **No external dependencies** - Works offline
2. ✅ **No deprecated APIs** - Future-proof solution
3. ✅ **Same functionality** - Generates QR codes for both address and BIS URL
4. ✅ **Error handling** - Graceful fallback if generation fails
5. ✅ **No server load** - Client-side generation

## Files Modified
- `wallet/themes/material/transactions_receive.html` - Updated QR code display
- `wallet/themes/common/js/qrcode.min.js` - Added QR code library

## Testing
The QR codes now generate properly in the browser without relying on external services. They maintain the same visual appearance and functionality as the original Google Charts implementation.