# Verifier

Module for managing Matrix device verification via the SAS (Short Authentication String) protocol. Lets you view your account's device list and verify them for use in encrypted rooms.

Just keep in mind that the bot verifies the device locally.

## Commands

### `.devices`

**Access:** OWNER

Displays a list of all devices linked to your Matrix account, along with their verification status.

```
.devices
# → 🔍 | Fetching devices...
# → 📱 | Your Devices:
#
#    🖥 | My Phone
#    └ QAZWSX | ❌ Unverified
#
#    🖥 | Bot Instance
#    └ EDCRFV | 🤖 (This Bot)
#
#    Use .verif [device_id] to verify a specific device.
```

**Statuses:**
- 🤖 (This Bot) — current device where the userbot is running
- ✅ Verified — device is verified, bot shares keys with it
- ❌ Unverified — device is not verified

### `.verif <device_id>`

**Access:** OWNER

Starts SAS verification for the specified device. Can't verify the bot's own device.

**Verification process:**

1. **Check** — the bot checks if the device exists and its current status
2. **Initiation** — sends a SAS request to the target device:
   ```
   🛡 | Verification initiated for: QAZWSX
   ⏳ Please accept the request on that device.
   ```
3. **Emoji** — after the request is accepted on the target device, the bot shows 7 emoji to compare:
   ```
   🔐 | SAS Emoji for device QAZWSX:
   
   🐶 | 🐱 | 🐭 | 🐹 | 🐰 | 🦊 | 🐻
   
   ⏳ Verification in progress...
   ```
4. **Waiting** — bot waits for the verification result (120 second timeout)
5. **Result**:
   - `success` — `✅ | Device QAZWSX verified successfully!`
   - `cancelled` — `❌ | Verification cancelled for device QAZWSX.`
   - `timeout` — `❌ | Verification failed for device QAZWSX.`

```
.verif QAZWSX
# → 🔍 | Checking device QAZWSX...
# → 🛡 | Verification initiated for: QAZWSX
#    ⏳ Please accept the request on that device.
# → 🔐 | SAS Emoji for device QAZWSX:
#    🐶 | 🐱 | 🐭 | 🐹 | 🐰 | 🦊 | 🐻
#    ⏳ Verification in progress...
# → ✅ | Device QAZWSX verified successfully!
```

## Status Check

Uses two verification levels:
1. `device.trust >= TrustState.VERIFIED` — direct status check from crypto store
2. `store.is_key_signed_by(target, signer)` — cross-signing check (device signed by master key)
