# TornadoWallet - Wallet Management Analysis

## Overview
This document explains how TornadoWallet manages wallet.json files, handles multiple addresses, and persists default address selection.

## Wallet.json Management

### File Location
- **Path**: `~/bismuth-private/wallet.json`
- **Directory**: Stored in user's home directory under `bismuth-private` folder
- **Access**: `helpers.get_private_dir()` returns the bismuth-private directory path

### Loading Multi-Wallet
**File**: `wallet/wallet.py:106`
```python
bismuth_client.load_multi_wallet("{}/wallet.json".format(wallet_dir))
```

**Structure**: The wallet.json contains multiple addresses accessible via:
```python
self.bismuth._wallet._addresses  # Array of address objects
```

## Multi-Wallet Address Management

### Address Array Access
**File**: `wallet/wallet.py:645`
```python
addresses = self.bismuth._wallet._addresses  # Array of all wallet addresses
```

### Address Selection Method
**File**: `wallet/wallet.py:668-684` (`load_address` method)
```python
async def load_address(self, params=None, post=False):
    """Set an address from the multiwallet as current address"""
    address = params[0]
    try:
        self.bismuth.set_address(address)  # Set as active address
    except Exception as e:
        # Handle error...
        return
    self.set_cookie("address", address)  # Persist selection in cookie
    self.redirect("/wallet/load")
```

### Address Limits
**File**: `wallet/wallet.py:844`
```python
if len(self.bismuth._wallet._data["addresses"]) >= self.settings["max_addresses"]:
    # Maximum addresses reached error
```

## Default Address Persistence

### Cookie-Based Storage
- **Active wallet**: `self.set_cookie("wallet", file_name)` (wallet.py:828)
- **Active address**: `self.set_cookie("address", address)` (wallet.py:683)

### Session Restoration
**File**: `wallet/modules/basehandlers.py:29-39`
```python
def initialize(self):
    # Load persisted wallet if needed
    wallet = self.get_cookie("wallet")
    
    # Restore last selected address
    address = self.get_cookie("address")
    if address:
        try:
            self.bismuth.set_address(address)  # Restore active address
        except:
            pass
```

### Default Route Configuration
**File**: `wallet/wallet.py:65`
```python
define("missing_address_route", default="/wallet/info", help="Route when no address is defined", type=str)
```

When no address is selected, the application redirects to `/wallet/info` (configurable).

## Key API Methods

### Core Address Management
1. **`bismuth.set_address(address)`** - Sets the currently active address
2. **`self.set_cookie("address", address)`** - Persists address selection in browser cookie
3. **`self.bismuth._wallet._addresses`** - Array containing all available addresses
4. **`self.bismuth.all_balances(for_display=True)`** - Gets balances for all addresses

### HTTP Routes
- **`/wallet/load_address/{address}`** - Route to select and set specific address as default
- **`/wallet/load`** - Main wallet loading page showing all addresses
- **`/wallet/load/detail`** - Detailed view with balances for each address
- **`/wallet/info`** - Wallet information page (default when no address selected)

## Implementation Flow

1. **Startup**: Application loads wallet.json using `load_multi_wallet()`
2. **Session Restore**: `BaseHandler.initialize()` checks cookies and restores last selected address
3. **Address Selection**: User selects address via `/wallet/load_address/{address}` route
4. **Persistence**: Selected address saved in browser cookie and set as active
5. **Future Sessions**: Cookie automatically restores the selected address

## Configuration Options

- **`max_addresses`**: Maximum number of addresses per wallet (default: 10)
- **`missing_address_route`**: Redirect route when no address is selected (default: "/wallet/info")
- **Wallet directory**: `~/bismuth-private/` (user's home directory)

## Example Usage

### Setting Default Address Programmatically
```python
# In a handler method
address = "your_bismuth_address_here"
self.bismuth.set_address(address)
self.set_cookie("address", address)
```

### Accessing All Addresses
```python
addresses = self.bismuth._wallet._addresses
for addr in addresses:
    print(f"Address: {addr['address']}")
```